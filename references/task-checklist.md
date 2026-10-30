# AutoEpicSeven Task Checklist

新增或重写一个任务时按此顺序执行。不要先搭大骨架；每次只完成一个用户可见能力。

## 1. Scope

- [ ] 任务名称、服务器和语言范围已确定
- [ ] 输入页面、输出页面和成功状态已确定
- [ ] 正常完成、今日已完成、资源不足、背包已满和网络异常已区分
- [ ] 所需 assets、OCR 字段、配置项和调度周期已列出
- [ ] 已阅读当前任务目录、相似 E7 任务和相关 git diff
- [ ] scheduler task、snake_case 入口、owner 目录和实际实现链已追踪

## 2. File Placement

- [ ] 在 `tasks/<owner>/` 下创建与功能同名的 `<feature>.py`
- [ ] `entry.py` 只在现有 owner 需要页面进入边界、入口 mixin 或实现分发时使用，不创建空入口层
- [ ] `run()` 只做编排，具体交互放在可独立测试的方法
- [ ] assets import 使用 `from ... import ...`
- [ ] 不修改生成的 `assets_*.py`
- [ ] 已读取 `offline-testing.md`（离线测试与验收），正式用例放当前工作树 `tests/`，旧副本保留且不再维护

## 3. Assets And Detection

- [ ] 截图分辨率为 1280x720
- [ ] asset name 使用有语义的 `UPPER_SNAKE_CASE`，未包含语言、坐标或截图序号
- [ ] 通用资源优先放 `share`，差异资源放对应服务器/语言目录
- [ ] 源图只放在根目录 `assets/`，预期生成 wrapper 路径已按 module 映射确认
- [ ] 切图前已按资源规范确定功能子目录；同名基础图、属性与多帧同目录，各语言模块路径对应一致
- [ ] 多模板使用 `.2` / `.3`，属性覆盖使用合法 attr 后缀且第一帧存在
- [ ] 固定位置亮灰判定优先 `match_color()`
- [ ] 先定位再判色使用 `match_template_color()`
- [ ] 只判断存在使用 `match_template()` 或 `match_template_luma()`
- [ ] 搜索区域足够小，模板具有区分度
- [ ] 已按 `asset-crop-tool.md`（切图用法）选择入口，核对预检路径、冲突、配方和隐私
- [ ] 修改源图后按完整模块定向生成包装，再用真实截图验证正反例；未运行删除旧文件的全量生成
- [ ] 缺少源图时先查可靠截图并用切图工具制作；没有来源才停止该分支，不写静默回退
- [ ] 涉及重复或闲置源图时，按 `asset-archive-tool.md`（素材暂存用法）核对依赖、预览、移动和校验，重新定向生成包装

## 4. Page Contract

- [ ] Page 有唯一 `check_button`
- [ ] 页面类型是正式页、popup 还是 overlay 已明确
- [ ] 国服/国际服存在性和 assets namespace 已明确
- [ ] 入口、BACK、关闭键、页内按钮的去向已连边
- [ ] `dynamic_return_button`、`shared_toolbar`、后台托管限制已决定
- [ ] 入场后二段点击、周奖励、结算、网络弹窗有独立状态处理
- [ ] 仅单服务器存在的页面按服务器条件注册和连边
- [ ] 目标服务器和非目标服务器都完成 import 与寻路回归检查

## 5. State Loop

状态循环使用以下骨架：

```python
def _execute_feature(self, skip_first_screenshot=True):
    while 1:
        if skip_first_screenshot:
            skip_first_screenshot = False
        else:
            self.device.screenshot()

        # End: positive, side-effect-free states only.
        if self.appear(FEATURE_COMPLETE):
            return True
        if self.appear(FEATURE_ALREADY_DONE):
            return False

        # Actions: every click must be retryable.
        if self.appear_then_click(FEATURE_ENTRY, interval=2):
            continue
        if self.appear_then_click(FEATURE_CONFIRM, interval=2):
            continue

        # Shared handlers run after feature-specific states.
        if self.handle_popup_confirm():
            continue
        if self.handle_network_error():
            continue
```

- [ ] 循环顶部截图，不在尾部截图
- [ ] 独立调用前已有首帧；离线用公共样本注入，手动实机才获取设备截图
- [ ] 退出条件在点击条件之前
- [ ] 退出只使用正面状态，不使用 `not appear(...)`
- [ ] 不使用 `appear_then_click()` 或 handler 直接退出
- [ ] 点击有合理 interval，纯退出识别没有 interval
- [ ] 没有用 sleep 推进状态
- [ ] `handle_*` 仅在执行操作时返回 True
- [ ] 状态变量在所有状态变化动作后正确重置
- [ ] 脆弱门控和状态变量有详尽英文注释

## 6. OCR

- [ ] 从 `Emulator_GameLanguage` 读取 OCR 语言，auto/空值回退 cn
- [ ] 每个 OCR 类显式传 `lang=lang`
- [ ] 没有把 `server.lang` 或 `global_cn` 传给 OCR
- [ ] 原始 OCR 文本保留在调试日志
- [ ] 格式、范围和 E7 业务语义都有校验
- [ ] 没有用会制造“格式合法、语义错误”数据的宽泛 fallback
- [ ] Counter 允许当前值合法超过恢复上限，除非该字段明确是硬上限

## 7. Config And Scheduler

- [ ] `task.yaml` 定义任务与选项组，并包含 `Scheduler`
- [ ] `argument.yaml` 定义选项、默认值和校验
- [ ] `gui.yaml` 定义额外 GUI 文本
- [ ] `default.yaml` / `override.yaml` 按需要更新
- [ ] `config_manual.py` 中调度优先级合理
- [ ] 当前任务入口加到 `aes.py`
- [ ] 已确认 TaskName 经 `inflection.underscore()` 后与 `aes.py` 方法名一致
- [ ] 配置属性使用 `Group_Argument` 扁平名称
- [ ] 所有支持语言翻译完整，没有占位路径
- [ ] 已按配置显隐流程区分固定隐藏、字段联动和任务入口，核对现有机制与对应验证（详见主文件直达的配置显隐章节）
- [ ] 涉及活动时，时间表、分服支持与选项登记一致；图形界面使用当前配置，隐藏不改原值或任务调度
- [ ] 领奖复查区分暂无奖励与当天全领；归档核对各服日期、当前注册及历史记录（见主文件直达的活动维护参考）
- [ ] 运行 config updater 前已请求用户批准

调度顺序默认遵循：重启、短时收取、每日奖励、日常战斗、常规战斗、纯消磨时间。活动收益很高不等于可以绕过页面、服务器和资源安全检查；优先级变化要结合活动有效期和失败影响决定。

## 8. Failure Policy

- [ ] 可预期子状态有局部 Timer 或重试上限
- [ ] 战斗和长动画没有不合理的短超时
- [ ] 人工处理状态抛 `RequestHumanTakeover`
- [ ] 不可恢复业务矛盾抛项目现有业务异常
- [ ] 框架 stuck / too-many-click 保护仍生效
- [ ] “今天已完成”与“脚本失败”没有混为同一个返回值或调度结果

## 9. Tests

- [ ] 问题截图或最小输入与预期结果已保存，主要识别和文字解析分支有正反例
- [ ] 状态循环通过公共辅助注入首帧，模拟点击不自动推进画面；有动作与禁止动作断言
- [ ] 覆盖首次进入、今日已完成、按钮点击未生效和弹窗打断
- [ ] 分服页面在独立子进程覆盖支持与不支持组合，业务导入前初始化服务器
- [ ] 稀有中间状态没有成为唯一完成或计数信号
- [ ] 分类、样本引用与运行器白名单同步，列出命令能找到新用例
- [ ] 使用当前工作树解释器和统一入口，所选测试全部通过且零跳过
- [ ] 缺图、损坏、缺模型与误建真实设备不会被静默忽略；不读取真实账号配置
- [ ] 失败日志、预期与实际及关联样本可追溯，未验证能力明确记录
- [ ] 权限失败时请求批准后重跑

## 10. Verification

1. 对改动 Python 文件运行定向 ruff。
2. 按 `offline-testing.md`（离线测试与验收）运行单用例和相关集；基础设施修改再验证默认全集与相关负向契约。
3. 运行 `git diff --check`。
4. 检查素材清理已通过暂存工具移动并校验，其他文件未删除，没有手改生成文件；正式测试已登记，历史目录、临时产物、真实配置及秘密信息没有入库。
5. 检查每个新 asset 的源图、生成 wrapper 和 import 路径能够互相追溯。
6. 实机验证仅按用户授权手动进行；从受支持页面和任务调度入口检查日志与结果截图，不接入自动测试。
7. 验收记录区分本机与云端，覆盖率注明业务范围；计划能力或待用户安装、推送的事项不写成已完成。

完成标准不是“理想路径点通”，而是按钮偶尔点击失败、截图偶尔损坏、设备变慢时，流程仍能通过下一帧状态继续推进或明确失败。
