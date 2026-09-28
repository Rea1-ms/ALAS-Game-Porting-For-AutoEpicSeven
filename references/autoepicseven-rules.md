# AutoEpicSeven Repository Rules

本文件是 AutoEpicSeven 开发的强制约束。执行任何代码修改前都要读取；与一般 ALAS 习惯冲突时，以本文件和用户当前指令为准。

## 运行环境

- 本机 Windows 11（操作系统）、PowerShell 7（终端）；离线云端当前使用 Ubuntu 24.04（操作系统）。解释器及测试依赖以当前工作树锁定记录为准。
- 所有文本使用 UTF-8。
- 所有换行使用 LF (`\n`)。
- 禁止删除文件，只允许修改或新增。
- 不使用破坏性 git 命令，不覆盖用户已有改动。

## Worktree 与目录

- “原始文件夹”指创建当前 worktree 的主检出目录，也就是当前仓库 git common dir 所在 `.git` 目录的父目录。先解析 `git rev-parse --git-common-dir` 的结果再定位，不在 skill 中硬编码盘符、用户名或机器路径；当前仓库不是 linked worktree 时，原始文件夹就是当前仓库根目录。
- 代码修改必须发生在当前 worktree，不得写回原始文件夹的业务代码。
- 切图工具目前将手动采样保存到主检出的 `test/screenshots/`（采样目录）；这只是手动工具的既有输出，正式测试仍在当前工作树维护。执行前核对预检路径与权限，不借此写入其他业务代码。
- 当前工作树 `tests/` 保存可自动断言的离线测试和精选脱敏样本；原 `test/` 保持忽略，保留迁移记录及手动工具，不再作为新增正式用例的维护位置。
- 开始前检查 `git status --short` 和相关 diff；遇到用户已有改动时在其基础上继续，不还原。

## 提交信息

为当前项目拟写或执行提交时，先查看 `git status --short`（工作树状态）、`git diff --cached`（暂存差异）和近期提交。尚未暂存时读取明确拟提交文件的差异；不为拟写标题而自动暂存。说明必须对应最终准备提交的内容，范围变化后重写，不能照搬聊天计划或历史草稿。

### 标题

默认格式为 `Type: summary`（类型、英文冒号、一个空格、英文动词短语）。沿用项目近期历史的首字母大写前缀；标题使用英文、动词原形开头，整行不超过 72 字符，不加句号或表情。用户明确指定语言或格式时按本次要求处理。

| 类型 | 使用场景 |
| --- | --- |
| `Feat`（功能） | 新增用户可用能力。 |
| `Fix`（修复） | 修正错误行为；同时增加回归用例仍以修复为主。 |
| `Perf`（性能） | 明确改善运行开销或速度。 |
| `Refactor`（重构） | 改变实现组织，保持外部行为。 |
| `Test`（测试） | 新增或调整测试、样本、回放和测试辅助。 |
| `Docs`（文档） | 只改变说明、规范或技能文档。 |
| `CI`（持续集成） | 云端工作流、执行环境及其必要依赖适配。 |
| `Chore`（维护） | 不属于以上类型的工具、依赖或工程维护。 |
| `Revert`（撤回） | 撤回既有提交，并说明对应提交及原因。 |

按最终改动的主要目的选一个类型，标题说明“什么行为改变了”，不罗列文件名，不写笼统的“更新代码”“修复问题”。历史中的 `Add`（新增）和 `Upd`（更新）保留，新提交默认使用表中更明确的类型；不为统一格式修改历史。

最小示例：`Fix: recognize uncollected items in exploration shops`（修复次元探查商店未收录商品识别）。中文释义用于解释示例，不属于实际提交标题。

### 正文与执行

- 简单改动只写标题；原因、行为变化或限制需要解释时，标题后空一行，正文简述问题、最终变化和相关验证。正文默认用中文，不重述探索过程和放弃的方案。
- 只写实际执行的验证及结果；未运行就说明原因。测试数量、覆盖范围和云端状态必须有记录，不能把局部通过写成全部通过。
- 一个提交围绕一个可解释的目的；业务修复及其回归测试放在一起，无关改动分开。实际提交前核对暂存文件和差异，不混入用户其他改动，不带入真实账号配置、`.env`（环境秘密文件）、日志或凭据。
- 提交和推送遵守当前授权及已有分工；只要求拟写提交说明不等于要求提交，用户自己推送时不要代推，也不重复索要已明确的授权。
- 多行提交说明先写入临时 UTF-8（统一编码）文本，再通过 `git commit -F <文件>`（从文件读取提交说明）传入，保留真实换行；不把正文拼进会解释变量或反引号的命令字符串。临时文件保留在忽略目录，不纳入提交。

## 编辑边界

- 不手改生成文件：
  - `module/config/argument/args.json`
  - `module/config/argument/menu.json`
  - `module/config/config_generated.py`
  - `tasks/*/assets/assets_*.py`
- assets import 一律使用 `from ... import ...`，与项目现有风格一致。
- 修改函数名、变量名或类名时，不自行执行全仓搜索替换。先完成可独立修改的代码，再汇总旧名到新名的映射，交给用户通过 IDE 批量重命名。
- 新增功能时，先在对应 `tasks/<task>/` 下建立与功能同名的实现文件，不把新逻辑继续堆入无关旧模块。
- 对反复修复、容易回归的状态变量、计数、时序门控和 fallback 必须写详尽英文注释。注释要说明：状态代表什么、何时重置、为什么不能按直觉简化、简化后会怎样失败。
- 页面跨越方法使用 Google 风格 docstring，并写 `Pages:` 的输入和输出状态。
- 不为了顺手整洁而重构当前任务之外的代码。

## 任务配置与入口

新增或修改任务时检查以下源文件：

- `module/config/argument/task.yaml`：任务包含哪些选项组，任务必须包含 `Scheduler`。
- `module/config/argument/argument.yaml`：选项组、选项、默认值、类型和校验。
- `module/config/argument/gui.yaml`：GUI 上额外出现的文本。
- `module/config/argument/default.yaml`：项目实际需要的默认任务配置。
- `module/config/argument/override.yaml`：锁定、隐藏或按服务器覆盖的值。
- `module/config/config_manual.py`：`SCHEDULER_PRIORITY` 调度顺序，修改后立即生效。
- `aes.py`：当前任务入口。`src.py` 是保留旧文件，不再作为新增 E7 任务的默认入口。

完成 YAML 修改后需要运行：

```powershell
.\.venv\Scripts\python.exe -m module.config.config_updater
```

运行这条命令前必须请求用户批准；不要先试跑，因为未批准时通常会出现权限错误。生成后检查所有支持语言的 i18n，不留下路径式占位文本。

配置访问使用扁平属性名：

```python
self.config.SpecialActivity_GetDailyReward
```

不要写嵌套属性：

```python
self.config.SpecialActivity.GetDailyReward
```

批量修改配置使用 `self.config.multi_set()`，任务结束使用 `task_delay(...)`，需要后续任务使用 `task_call(...)`。

## 测试

修改业务、识别、页面、测试基础设施或工作流前，读取 `offline-testing.md`（离线测试与验收）；详细命令、清单字段和验收步骤以该文件为准，实际接口以当前工作树代码为准。

- 正式用例在当前工作树 `tests/`（版本化目录），旧 `test/`（历史目录）、临时采样、日志和真实配置继续忽略。成熟测试与脱敏样本正常入库，不用强制添加忽略文件。
- 回归应覆盖问题输入和预期行为：识别改动验证正反例，流程改动验证连续帧、动作顺序和禁止动作；不要为可逆文档调整增加无意义测试。
- 新用例更新分类清单、样本引用及必要的运行器白名单，使用统一入口先列出、再执行；不批量导入历史测试。
- 本机使用当前工作树已有 `.venv/Scripts/python.exe`（虚拟环境解释器）；云端使用锁定的解释器，不改用 pytest（另一测试框架）或在沙箱中尝试 uv（环境管理工具）。用户负责安装时不自行安装。
- 首帧通过公共辅助注入；自动测试不实例化真实设备、不连接模拟器、不读取真实账号、不点击或消费。真实设备误调用必须失败。
- 准备条件缺失、断言失败和跳过不能视作通过；保留原断言和阈值，失败通过报告定位。环境、依赖、模型或权限不足时说明具体缺项，不换成不等价验证。
- 修改页面条件注册时，在独立子进程中检查国服、国际服中文和英文；导入业务前初始化服务器，肉鸽识别仍只声明支持国际服中文。
- 遵守已有安装、推送与实机操作分工。云端只有取得真实运行记录才写“已验证”，覆盖率只说明实际统计范围。

## Assets

- asset name、源图目录到 wrapper 的映射、frame/attr 后缀和入口定位以 `asset-task-conventions.md` 为唯一详细来源。
- 源素材使用 `dev_tools.asset_crop`（切图工具）制作，包装由按钮生成器生成；工具用法见 `asset-crop-tool.md`（切图说明）。源图片只放当前工作树根目录的 `assets/`，`tasks/<owner>/assets/` 只存放生成结果。
- 默认先运行已有图片的无界面预检，核对冲突后保存，再按完整模块路径定向生成。当前无过滤生成会删除旧文件，禁止作为默认命令执行。
- 生成的 `assets_*.py` 只能读取和 import，绝不能手改或放入源图片。
- 缺省 assets（识别资源）不能用异常捕获、动态查找或静默回退隐藏。先检查已有资源及用户截图，有可靠截图时使用切图工具补齐，没有来源才提醒用户补充画面。
- 同名元素只是不同服务器外观不同：在同一个 assets 定义中保留同名 `ButtonWrapper`，不支持的一侧使用 `None`，让页面层保持统一 import。
- 页面节点只存在于某个服务器：Page 注册、路由连边和入口逻辑都必须按服务器条件分开，不能只分 assets。
- 资源优先放 `share`。仅语言或服务器不同的图放 `cn`、`global_cn` 等对应目录。
- 短期只支持国际服中文的活动资源可放 `global_cn`；等国服同步后，再把相同资源迁回 `share` 并清理旧分支。清理涉及删除时只提醒用户，不自行删除。

## 颜色与模板匹配

按问题性质选择识别方式：

1. 固定位置、按钮不会漂、状态主要靠整体变灰或变亮：使用 `match_color()`。
2. 位置可能偏移，或同类元素很多，需要先确认“就是这个按钮”再判亮灰：使用 `match_template_color()`。
3. 只判断元素是否存在，不关心亮灰：使用 `match_template()` 或 `match_template_luma()`。

默认规则：

- 能用 `match_color()` 稳定解决，就不要上 `match_template_color()`。
- 只要存在“先定位再判色”，直接使用 `match_template_color()`。
- 模板要缩小到稳定且有区分度的区域；小而重复的模板容易产生密集多匹配。
- 多目标匹配后必须按可靠的界面几何关系配对，例如同一商品行的 Y 中心，而不是固定取第一个结果。

主界面与遮罩的双重判断示例：

```python
def is_in_main(self, interval=0):
    """
    Check whether the main page is visible without a popup overlay.

    The home-only star identifies the page. The fixed menu color then
    distinguishes a usable page from the same page dimmed by an overlay.
    """
    self.device.stuck_record_add(WHITE_STAR)

    if interval and not self.interval_is_reached(WHITE_STAR, interval=interval):
        return False

    appear = False
    if WHITE_STAR.match_template_luma(self.device.image):
        if POPUP_OVERLAY.match_color(self.device.image, threshold=30):
            appear = True

    if appear and interval:
        self.interval_reset(WHITE_STAR, interval=interval)

    return appear
```

这里模板负责确认主页，颜色负责确认没有遮罩；不要把复杂按钮的平均颜色判断误写成“定位按钮”的方案。

## OCR 语言

`server.lang` 是 assets 的命名空间，常见值为 `global_cn`、`global_en`；OCR 模型只接受项目规范化语言，如 `cn`、`en`、`jp`、`tw`，再映射到模型语言。两者不能混用。

统一流程：

1. 读取 `self.config.Emulator_GameLanguage`。
2. `auto`、空值或无效自动值回退到 `cn`。
3. 所有 `Ocr`、`Digit`、`DigitCounter`、`Duration` 都显式传入 `lang=lang`。
4. assets 继续由 `server.lang` 选择，不把 OCR 语言反向用于 assets。
5. OCR 结果必须做格式、范围和界面语义校验；保留原始 OCR 文本到调试日志。

最小示例：

```python
lang = self.config.Emulator_GameLanguage
if not lang or lang == 'auto':
    lang = 'cn'
ocr = Digit(OCR_VALUE, name='OCR_VALUE', lang=lang)
```

不要这样写：

```python
Digit(OCR_VALUE, lang=server.lang)
```

## 超时与异常

- 可预期、范围明确的子状态增加局部 Timer 或重试上限，例如“无免费次数”的短判定、确认弹窗出现窗口、滚动稳定检测。
- 长时间动画、战斗主循环不强制短超时。
- 最终由框架的 stuck 和 too-many-click 保护处理无法识别的死循环。
- 不可恢复的业务矛盾使用项目现有异常；需要人工处理的游戏状态使用 `RequestHumanTakeover`。

背包已满示例：

```python
if self.appear(PACKAGE_FULL, interval=0):
    logger.critical(
        'Combat: package full detected after COMBAT_START. '
        'Please clear inventory before starting combat again.'
    )
    raise RequestHumanTakeover
```

不要把资产缺失、程序错误或业务矛盾降级成“正常完成”。

## 页面要求

新增页面前必须明确至少五类信息：

1. 页面身份：页面名、唯一 `check_button`，以及正式页、popup 或 overlay 类型。
2. 服务器范围：国服是否存在、国际服是否存在；资源放 `share`、`cn` 还是 `global_cn`。
3. 路由关系：从哪些页面通过哪些入口进入；BACK、关闭键和页内按钮分别返回哪里。
4. 页面属性：是否需要 `dynamic_return_button`、`shared_toolbar`，是否无法进入后台托管战斗。
5. 入场状态：到达页面后是否还要点击一步；是否会被周奖励、结算、网络弹窗等打断。存在干扰就写独立状态循环，不能只依赖 `ui_goto`。

每个 Page 使用唯一识别状态。跨模块跳转优先 `ui_goto(page_xxx)`；只有调用者已经满足业务模块私有入口的页面前置条件时，才复用私有 enter 方法。

## 不同服务器分流

采用“公共流程 + 服务器分发”：

- 公共业务逻辑保持在一起，短期差异放在分服 assets 和少量分支方法。
- 判断服务器使用 `module/config/server.py` 的 `is_cn_server`、`is_oversea_server`，上层通常传 `self.config.Emulator_PackageName`。
- 服务器状态必须在依赖它的模块 import 之前完成初始化；条件页面注册尤其如此。
- 只有某服务器存在的 Page，必须按服务器条件注册和连边，否则另一侧的页面扫描与寻路会把不存在页面算进去。
- 模块入口按服务器和语言分发时，不支持的组合应正常记录并调度到下一周期，不应在 import 阶段崩溃。
- 不为了尚未支持的语言创建虚假页面占位或空 assets；支持范围以当前任务明确声明为准。

## 日志

- 日志需要让人能还原状态变化：任务阶段、关键配置、识别结果、计时器退出原因和最终状态。
- `logger.hr` 按任务、阶段、子阶段分级使用，避免每件事都强调。
- `logger.warning` 只用于确实需要关注但流程仍可继续的异常，不用于普通分支。
- 排查 OCR 时记录原始文本、候选区域和解析结果；排查模板时记录匹配数量和区域。
- 对一次性资源操作，失败时优先保存截图和上下文，不盲目重试完整流程。
