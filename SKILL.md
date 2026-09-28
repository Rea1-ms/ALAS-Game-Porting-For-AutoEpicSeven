---
name: alas-game-porting-for-autoepicseven
description: >
  Load for AutoEpicSeven/E7 automation development, porting, debugging, review, or tests: ALAS state loops, pages/routes, assets/cropping, OCR, server/language dispatch, config/scheduling, offline regression, screenshot replay, coverage, GitHub Actions, and commit messages. Also load for explicit $alas-game-porting-for-autoepicseven requests. Do not use for Epic Seven gameplay advice, unrelated Python/testing projects, or standalone Alasio/frontend migration.
---

# AutoEpicSeven ALAS Game Porting

Version: 0.2 (2026-09-27)

把 AutoEpicSeven 当作已经成熟的 E7 自动化项目直接开发。先读当前仓库代码、git 差异和本 skill 的参考资料。

## Always Read

- `references/autoepicseven-rules.md`：仓库硬约束、资产、OCR、页面、分服、测试与调度规则。
- 本文件的任务路由和状态循环底线。
- 修改业务、识别、页面、测试或工作流时，读取 `references/offline-testing.md`（离线测试与验收），再核对当前工作树的验收计划、统一入口与清单；用户没要求写测试也适用。

## Task Routing

| 当前工作 | 还要读取 |
| --- | --- |
| 新增或重写任务、交互流程 | `references/asset-task-conventions.md`、`references/task-checklist.md`、`references/state-loop-bible.md`、`references/development-lessons.md` |
| 审核状态循环、死循环、点击时序 | `references/state-loop-bible.md`、`references/development-lessons.md` |
| 新增页面、修改路由、按服务器注册页面 | `references/wiki-guidance.md` 的 Page/UI 与多服务器章节、`references/autoepicseven-rules.md` 的页面与分服章节、`references/development-lessons.md` 的页面与跨模块跳转章节 |
| 新增或调整 assets、命名、目录或颜色/模板识别 | `references/asset-task-conventions.md`、`references/wiki-guidance.md` 的识别章节、`references/development-lessons.md` 的识别经验 |
| 从截图切图、修改区域、多帧或配方重放 | `references/asset-crop-tool.md`（工具入口与用法）、`references/asset-task-conventions.md`；先预检再保存，定向生成包装 |
| 定位任务入口、实现文件或 asset 来源 | `references/asset-task-conventions.md` 的入口检索与反向定位章节 |
| OCR、资源栏、语言问题 | `references/autoepicseven-rules.md` 的 OCR 章节、`references/development-lessons.md` 的 OCR 经验 |
| 新增任务配置、GUI、调度入口 | `references/task-checklist.md` 的配置章节、`references/wiki-guidance.md` 的配置章节 |
| 修复识别、流程问题；新增测试、样本、回放或覆盖率 | `references/offline-testing.md`、`references/development-lessons.md`，按统一入口验证 |
| 测试依赖、云端工作流、失败报告或迁移验收 | `references/offline-testing.md` 的入口、报告及云端章节，核对当前工作树实际实现 |
| 拟写提交说明或准备提交 | `references/autoepicseven-rules.md#提交信息`，先核对实际差异与已有提交风格 |
| 维护或验收本 skill | `references/skill-evals.md` |
| 其他 AutoEpicSeven 开发 | 扫描所有参考文件标题，只读取与当前任务直接相关的章节 |

## Execution Defaults

1. 确认当前目录和 git worktree。代码只改当前 worktree，不向原始文件夹写业务代码。
2. 先读当前实现、相关 assets 调用和 git 差异，再定义输入页面、输出页面、成功状态、异常状态和服务器范围。
3. 新功能在当前 worktree（工作树）的 `tasks/<task>/` 下建立同名实现文件；正式离线用例和精选脱敏样本放当前工作树 `tests/` 并纳入版本控制。旧 `test/` 只保留历史副本及手动调试，继续忽略，不向其他工作树写正式测试；切图工具的手动采样按其现有路径保存。
4. 实现交互时使用截图优先的单层状态循环；顺序固定为退出状态、点击/动作状态、通用处理器。
5. 新增或调整源图先读 `references/asset-crop-tool.md`，已有截图默认使用无界面切图预检，再按模块生成包装。缺少可靠截图才停止并提醒用户补充；不隐藏缺失资产，不手改包装文件，不运行会删除旧文件的全量生成。
6. 配置只改 YAML 源文件和 `config_manual.py`。运行 config updater 前必须请求用户批准。
7. 修改识别、规则或流程时复用或补充有意义的回归用例；更新分类、样本引用和必要的运行器白名单，先列出确认，再用统一入口运行。自动入口只用模拟设备，不读取真实账号配置。实机调试独立手动执行，遵守本次授权。
8. 修改后运行定向静态检查和差异格式检查。不能运行的验证要明确说明。
9. 拟写或执行提交时读取 `references/autoepicseven-rules.md#提交信息`，按最终差异使用大写类型前缀和英文动词短语；验证只写实际结果，遵守已有提交与推送分工。

## State Loop Baseline

- 禁止用 `click + sleep` 推进游戏流程。
- 循环首部必须获取新截图，并用 `skip_first_screenshot` 复用调用方已有截图。
- 独立调用前准备首帧：离线测试使用公共辅助注入模拟帧；手动实机运行才执行设备截图。
- 退出只使用正面、无副作用的状态判断；不要用负面条件、点击语句或 handler 作为退出判据。
- 点击动作通常设置 2/3/5 秒 interval；纯退出识别不设置 interval。
- `handle_*` 只返回 bool；仅当确实执行了会改变画面的操作时返回 True。
- 可预期的短子状态使用局部 Timer 或重试上限；长动画和战斗主循环不强加短超时。
- 默认保持一个顶层状态循环。确需嵌套时，子循环必须退出到父循环能够识别的状态，并写清原因。
- 反复修复、容易回归的状态变量和时序判断必须写详尽英文注释，说明不变量及失败模式。

完整原文与反例见 `references/state-loop-bible.md`。

## Known Gotchas

- 切图工具的实机入口即使预检也会连接设备；采样目录不是正式测试集，生成包装默认按模块过滤。
  -> `references/asset-crop-tool.md`

- 新测试文件不会仅因位于 `tests/` 就进入默认集；必须登记白名单并通过列出命令确认。缺样本或模型不能跳过，断言不能放宽。
  -> `references/offline-testing.md#3-新增或修复一个问题`
- 模拟点击不代表画面变化或付款成功；重复帧、余额延迟和禁止动作都要明确断言。
  -> `references/offline-testing.md#4-样本和回放`
- 本机通过或写好工作流不等于云端已验证；覆盖率只声明实际统计范围。
  -> `references/offline-testing.md#7-覆盖率环境与云端`

- assets 语言与 OCR 模型语言是两套系统；`global_cn` 不能传给 OCR。
  -> `references/autoepicseven-rules.md#ocr-语言`
- 固定位置只判亮灰优先 `match_color()`；先定位再判色使用 `match_template_color()`。
  -> `references/autoepicseven-rules.md#颜色与模板匹配`
- 稀有中间状态不能用于业务计数。免费召唤应由“继续免费召唤/返回/已用完”等游戏状态推进。
  -> `references/development-lessons.md#召唤与活动流程`
- E7 的计数器当前值可以合法超过恢复上限，不要套用 `value <= total`。
  -> `references/development-lessons.md#ocr-与资源采集`
- 跨模块页面跳转优先 `ui_goto(Page)`，不要复用依赖隐含页面上下文的私有入口方法。
  -> `references/development-lessons.md#页面与跨模块跳转`
- 页面识别成功不等于入口已定位；同服多模板不会在直接点击时自动选中。排查导航错点时，先检查路由是否实际匹配入口。
  -> `references/development-lessons.md#导航直接点击与多模板入口`
- 修改函数名、变量名或类名时，不执行全仓批量替换；汇总重命名映射并交给用户用 IDE 处理。
  -> `references/autoepicseven-rules.md#编辑边界`

## Validation Loop

1. 运行针对改动文件的 ruff 检查；有错误就先修第一个真实问题，再重跑。
2. 运行 `git diff --check`，确认 LF、空白和补丁格式正常。
3. 按 `references/offline-testing.md` 运行统一入口中的目标用例与相关测试集；入口、依赖或工作流变更再验证默认全集及负向检查。准备失败不能改成跳过。测试需要权限时按实际限制请求批准；用户负责安装或推送时保留该分工。
4. 页面或分服注册有改动时，在独立子进程中检查国服、国际服中文和英文的导入、页面扫描与寻路；服务器在业务导入前初始化。
5. 最多连续修复三轮同类失败；仍失败时保留日志和截图，说明阻塞状态，不用宽泛 fallback 掩盖错误。

## Completion Checklist

- [ ] 当前 worktree、服务器范围、页面进出和成功条件已确认
- [ ] 没有删除文件，没有手改生成文件，没有隐藏缺失 assets
- [ ] 涉及切图时，已核对预检路径、冲突与样本隐私，定向生成包装，并用真实截图验证正反例
- [ ] 状态循环没有固定 sleep、负面退出或带副作用退出
- [ ] OCR 显式使用规范化语言，assets 仍由 server.lang 选择
- [ ] 页面节点、路由、返回关系和入场后弹窗状态已覆盖
- [ ] 配置源、调度顺序、aes.py 入口和 i18n 按任务需要完成
- [ ] 对应正式用例在当前工作树 `tests/`，分类、样本和运行器登记一致，统一入口可列出并执行
- [ ] 所选用例全部通过且零跳过；失败可定位到预期、实际及相关样本，未验证项有明确记录
- [ ] 临时产物与账号配置保持忽略；覆盖率和云端结果没有超出实际证据
- [ ] 日志足以解释失败位置，脆弱状态有详尽英文注释
- [ ] 涉及提交时，说明与最终差异一致，标题符合提交信息规范，验证结果真实且未混入无关文件
