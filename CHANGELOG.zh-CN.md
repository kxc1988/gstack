# 变更日志

## [0.7.0] - 2026-03-18 — YC Office Hours

**`/office-hours` — 在编写代码前与 YC 合作伙伴坐下来交流。**

两种模式。如果您正在构建初创公司，您会获得六个来自 YC 产品评估的强制问题：需求现实、现状、绝望的具体性、最窄的楔子、观察与惊喜，以及未来适应性。如果您正在开发副业项目、学习编码或参加黑客马拉松，您会获得一个热情的头脑风暴伙伴，帮助您找到想法的最酷版本。

两种模式都会编写一个设计文档，直接输入到 `/plan-ceo-review` 和 `/plan-eng-review`。会话结束后，技能会反思它对您思考方式的观察 — 具体观察，而非泛泛赞美。

**`/debug` — 找到根本原因，而非症状。**

当某些东西坏了而您不知道为什么时，`/debug` 是您的系统调试器。它遵循铁律：没有根本原因调查，就没有修复。追踪数据流，匹配已知错误模式（竞态条件、nil 传播、陈旧缓存、配置漂移），并一次测试一个假设。如果 3 个修复失败，它会停止并质疑架构，而不是继续尝试。

## [0.6.4.1] - 2026-03-18

### 新增

- **技能现在可通过自然语言发现。** 所有 12 个缺少明确触发短语的技能现在都有了它们 — 说 "deploy this" 时 Claude 会找到 `/ship`，说 "check my diff" 时会找到 `/review`。遵循 Anthropic 的最佳实践："描述字段不是摘要 — 而是触发的时机。"

## [0.6.4.0] - 2026-03-17

### 新增

- **`/plan-design-review` 现在是交互式的 — 评分 0-10，修复计划。** 设计师不再产生带有字母等级的报告，而是像 CEO 和 Eng 审查一样工作：对每个设计维度评分 0-10，解释 10 分的样子，然后编辑计划以达到目标。每个设计选择一个 AskUserQuestion。输出是一个更好的计划，而不是关于计划的文档。
- **CEO 审查现在会请设计师参与。** 当 `/plan-ceo-review` 在计划中检测到 UI 范围时，它会激活设计和 UX 部分（第 11 节），涵盖信息架构、交互状态覆盖、AI 草率风险和响应式意图。对于深度设计工作，它推荐 `/plan-design-review`。
- **15 个技能中的 14 个现在有完整的测试覆盖（E2E + LLM 评判 + 验证）。** 为 10 个缺少 LLM 评判质量评估的技能添加了评估：ship、retro、qa-only、plan-ceo-review、plan-eng-review、plan-design-review、design-review、design-consultation、document-release、gstack-upgrade。为 gstack-upgrade 添加了真实的 E2E 测试（之前是 `.todo`）。将 design-consultation 添加到命令验证中。
- **二分提交风格。** CLAUDE.md 现在要求每个提交都是单一的逻辑更改 — 重命名与重写分开，测试基础设施与测试实现分开。

### 更改

- `/qa-design-review` 重命名为 `/design-review` — 现在 `/plan-design-review` 是计划模式，"qa-" 前缀令人困惑。在所有 22 个文件中更新。

## [0.6.3.0] - 2026-03-17

### 新增

- **每个触及前端代码的 PR 现在都会自动获得设计审查。** `/review` 和 `/ship` 对更改的 CSS、HTML、JSX 和视图文件应用 20 项设计清单。捕获 AI 草率模式（紫色渐变、3 列图标网格、通用英雄文案）、排版问题（正文 < 16px、黑名单字体）、可访问性差距（`outline: none`）和 `!important` 滥用。机械 CSS 修复会自动应用；设计判断调用会先询问您。
- **`gstack-diff-scope` 对分支中更改的内容进行分类。** 运行 `eval $(gstack-diff-scope main)` 并获取 `SCOPE_FRONTEND=true/false`、`SCOPE_BACKEND`、`SCOPE_PROMPTS`、`SCOPE_TESTS`、`SCOPE_DOCS`、`SCOPE_CONFIG`。设计审查使用它在仅后端 PR 上静默跳过。Ship 预检使用它在触及前端文件时推荐设计审查。
- **设计审查在审查准备仪表板中显示。** 仪表板现在区分 "LITE"（代码级，在 /review 和 /ship 中自动运行）和 "FULL"（通过 /plan-design-review 与 browse 二进制文件进行视觉审计）。两者都显示为设计审查条目。
- **设计审查检测的 E2E 评估。** 植入的 CSS/HTML 夹具带有 7 个已知反模式（Papyrus 字体、14px 正文、`outline: none`、`!important`、紫色渐变、通用英雄文案、3 列功能网格）。评估验证 `/review` 至少捕获 7 个中的 4 个。

## [0.6.2.0] - 2026-03-17

### 新增

- **计划审查现在像世界上最好的人一样思考。** `/plan-ceo-review` 应用来自 Bezos（单向门、第 1 天代理怀疑）、Grove（偏执扫描）、Munger（反转）、Horowitz（战时意识）、Chesky/Graham（创始人模式）和 Altman（杠杆痴迷）的 14 种认知模式。`/plan-eng-review` 应用来自 Larson（团队状态诊断）、McKinley（默认无聊）、Brooks（本质与偶然复杂性）、Beck（使变更容易）、Majors（在生产中拥有您的代码）和 Google SRE（错误预算）的 15 种模式。`/plan-design-review` 应用来自 Rams（减法默认）、Norman（时间范围设计）、Zhuo（原则性品味）、Gebbia（为信任而设计，故事板旅程）和 Ive（可见的关怀）的 12 种模式。
- **潜在空间激活，而非清单。** 认知模式通过命名框架和人物，让 LLM 利用其对他们实际思考方式的深刻了解。指令是 "内化这些，不要列举它们" — 使每次审查成为真正的视角转变，而不是更长的清单。

## [0.6.1.0] - 2026-03-17

### 新增

- **E2E 和 LLM 评判测试现在只运行您更改的内容。** 每个测试都声明它依赖的源文件。当您运行 `bun run test:e2e` 时，它会检查您的差异并跳过依赖项未被触及的测试。仅更改 `/retro` 的分支现在运行 2 个测试而不是 31 个。使用 `bun run test:e2e:all` 强制运行所有测试。
- **`bun run eval:select` 预览将运行哪些测试。** 在花费 API 积分之前，确切地看到您的差异会触发哪些测试。支持 `--json` 用于脚本和 `--base <branch>` 覆盖基准分支。
- **完整性护栏捕获被遗忘的测试条目。** 一个免费的单元测试验证 E2E 和 LLM 评判测试文件中的每个 `testName` 在 TOUCHFILES 映射中都有相应的条目。没有条目的新测试会立即失败 `bun test` — 不会有静默的总是运行的降级。

### 更改

- `test:evals` 和 `test:e2e` 现在基于差异自动选择（之前：全有或全无）
- 新增 `test:evals:all` 和 `test:e2e:all` 脚本用于显式完整运行

## 0.6.1 — 2026-03-17 — Boil the Lake

现在每个 gstack 技能都遵循**完整性原则**：当 AI 使边际成本接近零时，始终推荐完整实现。当选项 A 只多 70 行代码时，不再有 "选择 B，因为它有 90% 的价值"。

阅读理念：https://garryslist.org/posts/boil-the-ocean

- **完整性评分**：每个 AskUserQuestion 选项现在显示完整性评分（1-10），偏向于完整解决方案
- **双重时间估计**：工作量估计同时显示人类团队和 CC+gstack 时间（例如，"人类：~2 周 / CC：~1 小时"），并带有任务类型压缩参考表
- **反模式示例**：前言中的具体 "不要这样做" 图库，使原则不抽象
- **首次使用入职**：新用户看到一次性介绍，链接到文章，可选择在浏览器中打开
- **审查完整性差距**：`/review` 现在标记完整版本成本 <30 分钟 CC 时间的捷径实现
- **Lake Score**：CEO 和 Eng 审查完成摘要显示有多少建议选择了完整选项与捷径
- **CEO + Eng 审查双重时间**：时间询问、工作量估计和愉悦机会都显示人类和 CC 时间尺度

## 0.6.0.1 — 2026-03-17

- **`/gstack-upgrade` 现在自动捕获过时的供应商副本。** 如果您的全局 gstack 是最新的，但项目中的供应商副本落后，`/gstack-upgrade` 会检测到不匹配并同步它。不再需要手动询问 "我们是否供应商它？" — 它会告诉您并提供更新。
- **升级同步更安全。** 如果 `./setup` 在同步供应商副本时失败，gstack 会从备份中恢复以前的版本，而不是留下损坏的安装。

### 对于贡献者

- 独立使用部分在 `gstack-upgrade/SKILL.md.tmpl` 中现在引用步骤 2 和 4.5（DRY），而不是复制检测/同步 bash 块。添加了一个新的版本比较 bash 块。
- 独立模式下的更新检查回退现在匹配前言模式（全局路径 → 本地路径 → `|| true`）。

## 0.6.0 — 2026-03-17

- **100% 测试覆盖率是良好氛围编码的关键。** 当您的项目没有测试框架时，gstack 现在会从头引导测试框架。检测您的运行时，研究最佳框架，让您选择，安装它，为您的实际代码编写 3-5 个真实测试，设置 CI/CD（GitHub Actions），创建 TESTING.md，并在 CLAUDE.md 中添加测试文化说明。之后的每个 Claude Code 会话都会自然地编写测试。
- **每个错误修复现在都有回归测试。** 当 `/qa` 修复错误并验证时，第 8e.5 阶段会自动生成一个回归测试，捕获确切的破坏场景。测试包括完整的归因追踪，可追溯到 QA 报告。自动递增的文件名防止会话之间的冲突。
- **自信发布 — 覆盖率审计显示什么被测试，什么没有。** `/ship` 第 3.4 步从您的差异构建代码路径图，搜索相应的测试，并生成带有质量星级的 ASCII 覆盖率图（★★★ = 边缘情况 + 错误，★★ = 愉快路径，★ = 冒烟测试）。差距会自动生成测试。PR 正文显示 "Tests: 42 → 47 (+5 new)"。
- **您的回顾跟踪测试健康。** `/retro` 现在显示总测试文件、此期间添加的测试、回归测试提交和趋势增量。如果测试比例低于 20%，它会将其标记为增长区域。
- **设计审查也会生成回归测试。** `/qa-design-review` 第 8e.5 阶段跳过纯 CSS 修复（这些由重新运行设计审计捕获），但为 JavaScript 行为更改（如下拉菜单损坏或动画失败）编写测试。

### 对于贡献者

- 添加了 `generateTestBootstrap()` 解析器到 `gen-skill-docs.ts`（~155 行）。在 RESOLVERS 映射中注册为 `{{TEST_BOOTSTRAP}}`。插入到 qa、ship（第 2.5 步）和 qa-design-review 模板中。
- 第 8e.5 阶段回归测试生成添加到 `qa/SKILL.md.tmpl`（46 行）和 CSS 感知变体到 `qa-design-review/SKILL.md.tmpl`（12 行）。规则 13 修改为允许创建新测试文件。
- 第 3.4 步测试覆盖率审计添加到 `ship/SKILL.md.tmpl`（88 行），带有质量评分标准和 ASCII 图表格式。
- 测试健康跟踪添加到 `retro/SKILL.md.tmpl`：3 个新的数据收集命令、指标行、叙述部分、JSON 架构字段。
- `qa-only/SKILL.md.tmpl` 在未检测到测试框架时获得推荐说明。
- `qa-report-template.md` 获得带有延迟测试规范的回归测试部分。
- ARCHITECTURE.md 占位符表更新，包含 `{{TEST_BOOTSTRAP}}` 和 `{{REVIEW_DASHBOARD}}`。
- WebSearch 添加到 qa、ship、qa-design-review 的允许工具中。
- 26 个新的验证测试，2 个新的 E2E 评估（引导 + 覆盖率审计）。
- 2 个新的 P3 TODO：非 GitHub 提供商的 CI/CD，自动升级弱测试。

## 0.5.4 — 2026-03-17

- **工程审查现在始终是完整审查。** `/plan-eng-review` 不再要求您在 "大变更" 和 "小变更" 模式之间选择。每个计划都获得完整的交互式演练（架构、代码质量、测试、性能）。范围减少仅在复杂性检查实际触发时建议 — 而不是作为常设菜单选项。
- **Ship 一旦您回答就不再询问审查。** 当 `/ship` 询问缺少的审查并且您说 "无论如何发布" 或 "不相关" 时，该决定会为分支保存。不再在预登陆修复后每次重新运行 `/ship` 时被重新询问。

### 对于贡献者

- 从 `plan-eng-review/SKILL.md.tmpl` 中删除了 SMALL_CHANGE / BIG_CHANGE / SCOPE_REDUCTION 菜单。范围减少现在是主动的（由复杂性检查触发）而不是菜单项。
- 向 `ship/SKILL.md.tmpl` 添加了审查门覆盖持久性 — 将 `ship-review-override` 条目写入 `$BRANCH-reviews.jsonl`，以便后续的 `/ship` 运行跳过门。
- 更新了 2 个 E2E 测试提示以匹配新流程。

## 0.5.3 — 2026-03-17

- **您始终掌控 — 即使在大胆梦想时。** `/plan-ceo-review` 现在将每个范围扩展呈现为您选择加入的个人决定。EXPANSION 模式热情推荐，但您对每个想法说 yes 或 no。不再有 "代理发狂并添加了 5 个您未要求的功能"。
- **新模式：SELECTIVE EXPANSION。** 将当前范围作为基线，但查看其他可能的内容。代理一个接一个地展示扩展机会，带有中立的建议 — 您挑选值得做的。完美用于现有功能的迭代，您既需要严谨又想被相邻改进所诱惑。
- **您的 CEO 审查愿景被保存，不会丢失。** 扩展想法、精选决策和 10x 愿景现在以结构化设计文档的形式持久化到 `~/.gstack/projects/{repo}/ceo-plans/`。过时的计划会自动归档。如果愿景异常出色，您可以将其提升到仓库中的 `docs/designs/` 供团队使用。

- **更智能的发布门。** `/ship` 不再在 CEO 和设计审查不相关时唠叨您。Eng Review 是唯一必需的门（您甚至可以通过 `gstack-config set skip_eng_review true` 禁用它）。CEO Review 推荐用于大型产品更改；Design Review 用于 UI 工作。仪表板仍然显示所有三个 — 它只是不会为可选的门阻止您。

### 对于贡献者

- 向 `plan-ceo-review/SKILL.md.tmpl` 添加了 SELECTIVE EXPANSION 模式，带有精选仪式、中立推荐姿态和 HOLD SCOPE 基线。
- 重写了 EXPANSION 模式的第 0D 步，包括选择加入仪式 — 将愿景提炼为离散提案，每个作为 AskUserQuestion 呈现。
- 添加了 CEO 计划持久性（0D-POST 步）：带有 YAML 前言的结构化 markdown（`status: ACTIVE/ARCHIVED/PROMOTED`）、范围决策表、归档流程。
- 添加了 `docs/designs` 提升步骤到审查日志之后。
- 模式快速参考表扩展到 4 列。
- 审查准备仪表板：Eng Review 必需（可通过 `skip_eng_review` 配置覆盖），CEO/Design 可选，由代理判断。
- 新测试：CEO 审查模式验证（4 种模式、持久性、提升），SELECTIVE EXPANSION E2E 测试。

## 0.5.2 — 2026-03-17

- **您的设计顾问现在承担创意风险。** `/design-consultation` 不仅提出安全、连贯的系统 — 它明确分解 SAFE CHOICES（类别基线）与 RISKS（您的产品脱颖而出的地方）。您选择打破哪些规则。每个风险都附有为什么它有效以及它成本的理由。
- **在选择前查看全景。** 当您选择研究时，代理会浏览您空间中的真实站点，带有屏幕截图和可访问性树分析 — 而不仅仅是网络搜索结果。您在做出设计决策前看到外面有什么。
- **预览页面看起来像您的产品。** 预览页面现在渲染逼真的产品模型 — 带有侧边导航和数据表的仪表板、带有英雄部分的营销页面、带有表单的设置页面 — 而不仅仅是字体样本和调色板。

## 0.5.1 — 2026-03-17
- **在发布前了解您的立场。** 每个 `/plan-ceo-review`、`/plan-eng-review` 和 `/plan-design-review` 现在将其结果记录到审查跟踪器中。每次审查结束时，您会看到一个**审查准备仪表板**，显示哪些审查已完成，何时运行，以及它们是否干净 — 带有明确的 CLEARED TO SHIP 或 NOT READY  verdict。
- **`/ship` 在创建 PR 前检查您的审查。** 预检现在读取仪表板并在审查缺失时询问您是否要继续。仅提供信息 — 它不会阻止您，但您会知道您跳过了什么。
- **少一件需要复制粘贴的事情。** SLUG 计算（从 git remote 计算 `owner-repo` 的不透明 sed 管道）现在是一个共享的 `bin/gstack-slug` 助手。模板中所有 14 个内联副本都替换为 `eval $(gstack-slug)`。如果格式 ever 更改，只需修复一次。
- **屏幕截图现在在 QA 和浏览会话中可见。** 当 gstack 拍摄屏幕截图时，它们现在显示为输出中的可点击图像元素 — 不再有您看不到的不可见 `/tmp/browse-screenshot.png` 路径。在 `/qa`、`/qa-only`、`/plan-design-review`、`/qa-design-review`、`/browse` 和 `/gstack` 中工作。

### 对于贡献者

- 向 `gen-skill-docs.ts` 添加了 `{{REVIEW_DASHBOARD}}` 解析器 — 共享仪表板阅读器注入到 4 个模板（3 个审查技能 + ship）中。
- 添加了 `bin/gstack-slug` 助手（5 行 bash），带有单元测试。输出 `SLUG=` 和 `BRANCH=` 行，将 `/` 清理为 `-`。
- 新 TODO：智能审查相关性检测（P3），用于审查门控 PR 合并的 `/merge` 技能（P2）。

## 0.5.0 — 2026-03-16

- **您的网站刚刚获得设计审查。** `/plan-design-review` 打开您的网站并像高级产品设计师一样审查它 — 排版、间距、层次结构、颜色、响应式、交互和 AI 草率检测。每个类别获得字母等级（A-F），双重标题 "设计分数" + "AI 草率分数"，以及不拐弯抹角的结构化第一印象。
- **它也可以修复它发现的问题。** `/qa-design-review` 运行相同的设计师眼睛审计，然后通过原子 `style(design):` 提交和前后屏幕截图迭代修复源代码中的设计问题。默认 CSS 安全，带有为样式更改调整的更严格的自我调节启发式。
- **了解您的实际设计系统。** 两个技能都通过 JS 提取您现场站点的字体、颜色、标题比例和间距模式 — 然后提供将推断的系统保存为 `DESIGN.md` 基线。最终了解您实际使用了多少种字体。
- **AI 草率检测是头条指标。** 每个报告以两个分数开头：设计分数和 AI 草率分数。AI 草率清单捕获 10 个最可识别的 AI 生成模式 — 3 列功能网格、紫色渐变、装饰性 blob、表情符号项目符号、通用英雄文案。
- **设计回归跟踪。** 报告写入 `design-baseline.json`。下次运行自动比较：每个类别的等级变化、新发现、已解决的发现。观察您的设计分数随时间提高。
- **80 项设计审计清单** 跨 10 个类别：视觉层次结构、排版、颜色/对比度、间距/布局、交互状态、响应式、运动、内容/微文案、AI 草率和性能即设计。从 Vercel 的 100+ 规则、Anthropic 的前端设计技能和 6 个其他设计框架中提取。

### 对于贡献者

- 向 `gen-skill-docs.ts` 添加了 `{{DESIGN_METHODOLOGY}}` 解析器 — 共享设计审计方法注入到 `/plan-design-review` 和 `/qa-design-review` 模板中，遵循 `{{QA_METHODOLOGY}}` 模式。
- 添加了 `~/.gstack-dev/plans/` 作为长期愿景文档的本地计划目录（不签入）。CLAUDE.md 和 TODOS.md 更新。
- 向 TODOS.md 添加了 `/setup-design-md`（P2）用于从头开始的交互式 DESIGN.md 创建。

## 0.4.5 — 2026-03-16

- **审查发现现在实际上得到修复，而不仅仅是列出。** `/review` 和 `/ship` 过去打印信息性发现（死代码、测试差距、N+1 查询）然后忽略它们。现在每个发现都得到行动：明显的机械修复自动应用，真正模糊的问题批量处理成单个问题而不是 8 个单独的提示。您会看到每个自动修复的 `[AUTO-FIXED] file:line Problem → what was done`。
- **您控制 "直接修复" 和 "先问我" 之间的界限。** 死代码、过时注释、N+1 查询会自动修复。安全问题、竞态条件、设计决策会浮现供您调用。分类位于一个地方（`review/checklist.md`），因此 `/review` 和 `/ship` 保持同步。

### 修复

- **`$B js "const x = await fetch(...); return x.status"` 现在工作。** `js` 命令过去将所有内容包装为表达式 — 因此 `const`、分号和多行代码都坏了。现在它检测语句并使用块包装器，就像 `eval` 已经做的那样。
- **点击下拉选项不再永远挂起。** 如果代理在快照中看到 `@e3 [option] "Admin"` 并运行 `click @e3`，gstack 现在自动选择该选项，而不是在不可能的 Playwright 点击上挂起。正确的事情就会发生。
- **当点击是错误的工具时，gstack 会告诉您。** 通过 CSS 选择器点击 `<option>` 过去会因神秘的 Playwright 错误而超时。现在您会得到：`"Use 'browse select' instead of 'click' for dropdown options."`

### 对于贡献者

- 门分类 → 严重性分类重命名（严重性决定显示顺序，而不是您是否看到提示）。
- 向 `review/checklist.md` 添加了 Fix-First Heuristic 部分 — 规范的 AUTO-FIX vs ASK 分类。
- 新验证测试：`Fix-First Heuristic exists in checklist and is referenced by review + ship`。
- 在 `read-commands.ts` 中提取了 `needsBlockWrapper()` 和 `wrapForEvaluate()` 助手 — 由 `js` 和 `eval` 命令共享（DRY）。
- 向 `BrowserManager` 添加了 `getRefRole()` — 为 ref 选择器暴露 ARIA 角色，而不改变 `resolveRef` 返回类型。
- 点击处理程序自动将 `[role=option]` refs 路由到通过父 `<select>` 的 `selectOption()`，带有 DOM `tagName` 检查以避免阻塞自定义列表框组件。
- 6 个新测试：多行 js、分号、语句关键字、简单表达式、选项自动路由、CSS 选项错误指导。

## 0.4.4 — 2026-03-16

- **新版本在不到一小时内检测到，而不是半天。** 更新检查缓存设置为 12 小时，这意味着您可能一整天都停留在旧版本，而新版本不断发布。现在 "您是最新的" 在 60 分钟后过期，因此您会在一小时内看到升级。"升级可用" 仍然唠叨 12 小时（这是重点）。
- **`/gstack-upgrade` 总是进行真实检查。** 直接运行 `/gstack-upgrade` 现在绕过缓存并对 GitHub 进行新鲜检查。不再有 "您已经在最新版本" 而实际上不是。

### 对于贡献者

- 拆分 `last-update-check` 缓存 TTL：`UP_TO_DATE` 为 60 分钟，`UPGRADE_AVAILABLE` 为 720 分钟。
- 向 `bin/gstack-update-check` 添加了 `--force` 标志（在检查前删除缓存文件）。
- 3 个新测试：`--force` 破坏 UP_TO_DATE 缓存，`--force` 破坏 UPGRADE_AVAILABLE 缓存，60 分钟 TTL 边界测试带有 `utimesSync`。

## 0.4.3 — 2026-03-16

- **新 `/document-release` 技能。** 在 `/ship` 之后但在合并之前运行它 — 它读取项目中的每个文档文件，交叉引用差异，并更新 README、ARCHITECTURE、CONTRIBUTING、CHANGELOG 和 TODOS 以匹配您实际发布的内容。风险更改会以问题形式浮现；其他一切都是自动的。
- **每个问题现在都清晰明了，每次都如此。** 您过去需要运行 3+ 会话，gstack 才会给您完整的上下文和简单的英语解释。现在每个问题 — 即使在单个会话中 — 都会告诉您项目、分支和正在发生的事情，解释得足够简单，以便在上下文切换中理解。不再有 "抱歉，用更简单的方式解释给我听。"
- **分支名称始终正确。** gstack 现在在运行时检测您当前的分支，而不是依赖会话开始时的快照。在会话中途切换分支？gstack 会跟上。

### 对于贡献者

- 将 ELI16 规则合并到基本 AskUserQuestion 格式中 — 一种格式而不是两种，没有 `_SESSIONS >= 3` 条件。
- 向前言 bash 块添加了 `_BRANCH` 检测（`git branch --show-current` 带有回退）。
- 添加了分支检测和简化规则的回归保护测试。

## 0.4.2 — 2026-03-16

- **`$B js "await fetch(...)"` 现在正常工作。** `$B js` 或 `$B eval` 中的任何 `await` 表达式都会自动包装在异步上下文中。不再有 `SyntaxError: await is only valid in async functions`。单行 eval 文件直接返回值；多行文件使用显式 `return`。
- **贡献者模式现在反映，而不仅仅是反应。** 贡献者模式不再只在出现问题时提交报告，而是现在定期提示反思："将您的 gstack 体验评为 0-10。不是 10？想想为什么。" 捕获被动检测错过的生活质量问题和摩擦。报告现在包括 0-10 评级和 "什么会使这成为 10" 以专注于可操作的改进。
- **技能现在尊重您的分支目标。** `/ship`、`/review`、`/qa` 和 `/plan-ceo-review` 检测您的 PR 实际目标的分支，而不是假设 `main`。堆叠分支、针对功能分支的 Conductor 工作区以及使用 `master` 的仓库现在都正常工作。
- **`/retro` 在任何默认分支上工作。** 使用 `master`、`develop` 或其他默认分支名称的仓库会自动检测 — 不再因为分支名称错误而导致空回顾。
- **新 `{{BASE_BRANCH_DETECT}}` 占位符** 为技能作者 — 将其放入任何模板中，免费获得 3 步分支检测（PR 基础 → 仓库默认 → 回退）。
- **3 个新的 E2E 冒烟测试** 验证基础分支检测在 ship、review 和 retro 技能中端到端工作。

### 对于贡献者

- 添加了 `hasAwait()` 助手，带有注释剥离以避免在 eval 文件中对 `// await` 的误报。
- 智能 eval 包装：单行 → 表达式 `(...)`，多行 → 块 `{...}` 带有显式 `return`。
- 6 个新的异步包装单元测试，40 个新的贡献者模式前言验证测试。
- 校准示例以历史方式构建（"过去失败"），以避免在修复后暗示活 bug。
- 向 CLAUDE.md 添加了 "编写 SKILL 模板" 部分 — 关于自然语言而非 bash 风格、动态分支检测、自包含代码块的规则。
- 硬编码主分支回归测试扫描所有 `.tmpl` 文件以查找带有硬编码 `main` 的 git 命令。
- QA 模板清理：删除 `REPORT_DIR` shell 变量，将端口检测简化为 prose。
- gstack-upgrade 模板：bash 块之间变量引用的显式跨步骤 prose。

## 0.4.1 — 2026-03-16

- **gstack 现在注意到它搞砸了。** 开启贡献者模式（`gstack-config set gstack_contributor true`），gstack 会自动写出出错的地方 — 您在做什么，什么坏了，重现步骤。下次有什么烦到您时，错误报告已经写好了。分叉 gstack 并自己修复它。
- **同时处理多个会话？gstack 会跟上。** 当您打开 3+ gstack 窗口时，每个问题现在都会告诉您哪个项目、哪个分支以及您在做什么。不再盯着问题想 "等等，这是哪个窗口？"
- **每个问题现在都带有建议。** 不是向您倾销选项并让您思考，gstack 告诉您它会选择什么以及为什么。每个技能都采用相同的清晰格式。
- **/review 现在捕获被遗忘的枚举处理程序。** 添加新的状态、层级或类型常量？/review 会跟踪它通过代码库中的每个 switch 语句、允许列表和过滤器 — 不仅仅是您更改的文件。在它们发布前捕获 "添加了值但忘记处理它" 类别的 bug。

### 对于贡献者

- 将所有 11 个技能模板中的 `{{UPDATE_CHECK}}` 重命名为 `{{PREAMBLE}}` — 一个启动块现在处理更新检查、会话跟踪、贡献者模式和问题格式。
- DRY'd plan-ceo-review 和 plan-eng-review 问题格式，引用前言基线而不是重复规则。
- 向 CLAUDE.md 添加了 CHANGELOG 风格指南和供应商符号链接感知文档。

## 0.4.0 — 2026-03-16

### 新增
- **QA-only 技能** (`/qa-only`) — 仅报告 QA 模式，发现并记录 bug 而不进行修复。向您的团队移交干净的 bug 报告，而无需代理触摸您的代码。
- **QA 修复循环** — `/qa` 现在运行查找-修复-验证周期：发现 bug，修复它们，提交，重新导航以确认修复生效。一个命令从损坏到发布。
- **计划到 QA 工件流** — `/plan-eng-review` 写入测试计划工件，`/qa` 自动拾取。您的工程审查现在直接输入到 QA 测试中，无需手动复制粘贴。
- **`{{QA_METHODOLOGY}}` DRY 占位符** — 共享 QA 方法块注入到 `/qa` 和 `/qa-only` 模板中。当您更新测试标准时，保持两个技能同步。
- **评估效率指标** — 所有评估表面现在显示轮次、持续时间和成本，带有自然语言 **Takeaway** 评论。一目了然地查看您的提示更改是否使代理更快或更慢。
- **`generateCommentary()` 引擎** — 解释比较差异，所以您不必：标记回归，注意改进，并产生整体效率摘要。
- **评估列表列** — `bun run eval:list` 现在显示每次运行的轮次和持续时间。立即发现昂贵或缓慢的运行。
- **评估摘要每测试效率** — `bun run eval:summary` 显示跨运行的每个测试的平均轮次/持续时间/成本。确定哪些测试随时间花费最多。
- **`judgePassed()` 单元测试** — 提取并测试通过/失败判断逻辑。
- **3 个新的 E2E 测试** — qa-only 无修复护栏，带有提交验证的 qa 修复循环，plan-eng-review 测试计划工件。
- **浏览器 ref 陈旧检测** — `resolveRef()` 现在检查元素计数以检测页面突变后的陈旧 refs。SPA 导航不再导致缺失元素的 30 秒超时。
- 3 个新的快照测试用于 ref 陈旧。

### 更改
- QA 技能提示以显式两循环工作流（查找 → 修复 → 验证）重新结构化。
- `formatComparison()` 现在显示每个测试的轮次和持续时间差异以及成本。
- `printSummary()` 显示轮次和持续时间列。
- `eval-store.test.ts` 修复了预先存在的 `_partial` 文件断言错误。

### 修复
- 浏览器 ref 陈旧 — 页面突变（例如 SPA 导航）前收集的 refs 现在被检测并重新收集。消除了动态站点上一类不稳定的 QA 失败。

## 0.3.9 — 2026-03-15

### 新增
- **`bin/gstack-config` CLI** — `~/.gstack/config.yaml` 的简单 get/set/list 接口。由 update-check 和 upgrade 技能用于持久设置（auto_upgrade, update_check）。
- **智能更新检查** — 12 小时缓存 TTL（之前为 24 小时），用户拒绝升级时的指数式暂停回退（24 小时 → 48 小时 → 1 周），`update_check: false` 配置选项完全禁用检查。新版本发布时暂停重置。
- **自动升级模式** — 在配置中设置 `auto_upgrade: true` 或 `GSTACK_AUTO_UPGRADE=1` 环境变量以跳过升级提示并自动更新。
- **4 选项升级提示** — "是，现在升级"，"始终保持我最新"，"现在不"（暂停），"永远不再问"（禁用）。
- **供应商副本同步** — `/gstack-upgrade` 现在在升级主安装后检测并更新当前项目中的本地供应商副本。
- 25 个新测试：11 个用于 gstack-config CLI，14 个用于 update-check 中的暂停/配置路径。

### 更改
- README 升级/故障排除部分简化，引用 `/gstack-upgrade` 而不是长粘贴命令。
- 升级技能模板版本提升到 v1.1.0，带有 `Write` 工具权限用于配置编辑。
- 所有 SKILL.md 前言都用新的升级流程描述更新。

## 0.3.8 — 2026-03-14

### 新增
- **TODOS.md 作为单一事实来源** — 将 `TODO.md`（路线图）和 `TODOS.md`（近期）合并到一个文件中，按技能/组件组织，具有 P0-P4 优先级排序和完成部分。
- **`/ship` 第 5.5 步：TODOS.md 管理** — 自动从差异中检测已完成的项目，用版本注释标记它们完成，提供在缺失或非结构化时创建/重组 TODOS.md。
- **跨技能 TODOS 感知** — `/plan-ceo-review`、`/plan-eng-review`、`/retro`、`/review` 和 `/qa` 现在读取 TODOS.md 以获取项目上下文。`/retro` 添加了待办事项健康指标（开放计数、P0/P1 项目、流失）。
- **共享 `review/TODOS-format.md`** — 由 `/ship` 和 `/plan-ceo-review` 引用的规范 TODO 项目格式，防止格式漂移（DRY）。
- **Greptile 2 级回复系统** — 第一层（友好，内联差异 + 解释）用于第一响应；第二层（坚定，完整证据链 + 重新排名请求）当 Greptile 在先前回复后重新标记时。
- **Greptile 回复模板** — `greptile-triage.md` 中的结构化模板，用于修复（内联差异）、已修复（已完成的操作）和误报（证据 + 建议重新排名）。替换模糊的单行回复。
- **Greptile 升级检测** — 显式算法检测评论线程上的先前 GStack 回复并自动升级到第 2 层。
- **Greptile 严重性重新排名** — 回复现在包括 `**Suggested re-rank:**` 当 Greptile 错误分类问题严重性时。
- 跨技能 `TODOS-format.md` 引用的静态验证测试。

### 修复
- **`.gitignore` 追加失败被静默吞没** — `ensureStateDir()` 裸露的 `catch {}` 替换为仅 ENOENT 静默；非 ENOENT 错误（EACCES, ENOSPC）记录到 `.gstack/browse-server.log`。

### 更改
- `TODO.md` 删除 — 所有项目合并到 `TODOS.md`。
- `/ship` 第 3.75 步和 `/review` 第 5 步现在引用 `greptile-triage.md` 中的回复模板和升级检测。
- `/ship` 第 6 步提交顺序在最终提交中包含 TODOS.md 以及 VERSION + CHANGELOG。
- `/ship` 第 8 步 PR 正文包含 TODOS 部分。

## 0.3.7 — 2026-03-14

### 新增
- **屏幕截图元素/区域裁剪** — `screenshot` 命令现在支持通过 CSS 选择器或 @ref 进行元素裁剪（`screenshot "#hero" out.png`，`screenshot @e3 out.png`），区域裁剪（`screenshot --clip x,y,w,h out.png`），以及仅视口模式（`screenshot --viewport out.png`）。使用 Playwright 的原生 `locator.screenshot()` 和 `page.screenshot({ clip })`。全页面仍然是默认值。
- 10 个新测试覆盖所有屏幕截图模式（视口、CSS、@ref、裁剪）和错误路径（未知标志、互斥、无效坐标、路径验证、不存在的选择器）。

## 0.3.6 — 2026-03-14

### 新增
- **E2E 可观察性** — 心跳文件（`~/.gstack-dev/e2e-live.json`），每运行日志目录（`~/.gstack-dev/e2e-runs/{runId}/`），progress.log，每测试 NDJSON 转录，持久失败转录。所有 I/O 非致命。
- **`bun run eval:watch`** — 实时终端仪表板每 1 秒读取心跳 + 部分评估文件。显示已完成的测试，当前测试带有轮次/工具信息，陈旧检测（>10 分钟），`--tail` 用于 progress.log。
- **增量评估保存** — `savePartial()` 在每个测试完成后写入 `_partial-e2e.json`。崩溃弹性：部分结果在被终止的运行中存活。永远不清理。
- **机器可读诊断** — 评估 JSON 中的 `exit_reason`、`timeout_at_turn`、`last_tool_call` 字段。启用 `jq` 查询用于自动修复循环。
- **API 连接预检查** — E2E 套件在消耗测试预算之前立即抛出 ConnectionRefused。
- **`is_error` 检测** — `claude -p` 可以在 API 失败时返回 `subtype: "success"` 和 `is_error: true`。现在正确分类为 `error_api`。
- **Stream-json NDJSON 解析器** — `parseNDJSON()` 纯函数，用于从 `claude -p --output-format stream-json --verbose` 实时 E2E 进度。
- **评估持久性** — 结果保存到 `~/.gstack-dev/evals/`，并与之前的运行自动比较。
- **评估 CLI 工具** — `eval:list`、`eval:compare`、`eval:summary` 用于检查评估历史。
- **所有 9 个技能转换为 `.tmpl` 模板** — plan-ceo-review、plan-eng-review、retro、review、ship 现在使用 `{{UPDATE_CHECK}}` 占位符。更新检查前言的单一事实来源。
- **3 层评估套件** — 第 1 层：静态验证（免费），第 2 层：通过 `claude -p` 的 E2E（~$3.85/运行），第 3 层：LLM 作为评判（~$0.15/运行）。由 `EVALS=1` 控制。
- **植入错误结果测试** — 带有已知错误的评估夹具，LLM 评判评分检测。
- 15 个可观察性单元测试，涵盖心跳架构、progress.log 格式、NDJSON 命名、savePartial、finalize、watcher 渲染、陈旧检测、非致命 I/O。
- 计划-ceo-review、plan-eng-review、retro 技能的 E2E 测试。
- 更新检查退出代码回归测试。
- `test/helpers/skill-parser.ts` — `getRemoteSlug()` 用于 git remote 检测。

### 修复
- **浏览器二进制发现对代理损坏** — 用 SKILL.md 设置块中的显式 `browse/dist/browse` 路径替换 `find-browse` 间接。
- **更新检查退出代码 1 误导代理** — 添加 `|| true` 以防止无更新可用时的非零退出。
- **browse/SKILL.md 缺少设置块** — 添加 `{{BROWSE_SETUP}}` 占位符。
- **plan-ceo-review 超时** — 在测试目录中初始化 git 仓库，跳过代码库探索，将超时增加到 420s。
- 植入错误评估可靠性 — 简化提示，降低检测基线，对 max_turns 薄片具有弹性。

### 更改
- **模板系统扩展** — `gen-skill-docs.ts` 中的 `{{UPDATE_CHECK}}` 和 `{{BROWSE_SETUP}}` 占位符。所有使用 browse 的技能从单一事实来源生成。
- 丰富了 14 个命令描述，包括特定的参数格式、有效值、错误行为和返回类型。
- 设置块首先检查工作区本地路径（用于开发），回退到全局安装。
- LLM 评估评判从 Haiku 升级到 Sonnet 4.6。
- `generateHelpText()` 从 COMMAND_DESCRIPTIONS 自动生成（替换手工维护的帮助文本）。

## 0.3.3 — 2026-03-13

### 新增
- **SKILL.md 模板系统** — 带有 `{{COMMAND_REFERENCE}}` 和 `{{SNAPSHOT_FLAGS}}` 占位符的 `.tmpl` 文件，在构建时从源代码自动生成。从结构上防止文档和代码之间的命令漂移。
- **命令注册表** (`browse/src/commands.ts`) — 所有 browse 命令的单一事实来源，带有类别和丰富的描述。零副作用，安全导入到构建脚本和测试中。
- **快照标志元数据** (`browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS` 数组) — 元数据驱动的解析器替换手工编码的 switch/case。在一个地方添加标志会更新解析器、文档和测试。
- **第 1 层静态验证** — 43 个测试：从 SKILL.md 代码块解析 `$B` 命令，针对命令注册表和快照标志元数据进行验证
- **第 2 层 E2E 测试** 通过 Agent SDK — 生成真实的 Claude 会话，运行技能，扫描 browse 错误。由 `SKILL_E2E=1` 环境变量控制（~$0.50/运行）
- **第 3 层 LLM 作为评判评估** — Haiku 对生成的文档进行清晰度/完整性/可操作性评分（阈值 ≥4/5），以及与手工维护基线的回归测试。由 `ANTHROPIC_API_KEY` 控制
- **`bun run skill:check`** — 健康仪表板，显示所有技能、命令计数、验证状态、模板新鲜度
- **`bun run dev:skill`** — 监视模式，在每个模板或源文件更改时重新生成并验证 SKILL.md
- **CI 工作流** (`.github/workflows/skill-docs.yml`) — 在 push/PR 上运行 `gen:skill-docs`，如果生成的输出与提交的文件不同则失败
- `bun run gen:skill-docs` 脚本用于手动重新生成
- `bun run test:eval` 用于 LLM 作为评判评估
- `test/helpers/skill-parser.ts` — 从 Markdown 中提取并验证 `$B` 命令
- `test/helpers/session-runner.ts` — Agent SDK 包装器，带有错误模式扫描和转录保存
- **ARCHITECTURE.md** — 设计决策文档，涵盖守护进程模型、安全性、ref 系统、日志记录、崩溃恢复
- **Conductor 集成** (`conductor.json`) — 工作区设置/拆除的生命周期钩子
- **`.env` 传播** — `bin/dev-setup` 自动将 `.env` 从主工作树复制到 Conductor 工作区
- `.env.example` 模板用于 API 密钥配置

### 更改
- 构建现在在编译二进制文件之前运行 `gen:skill-docs`
- `parseSnapshotArgs` 是元数据驱动的（迭代 `SNAPSHOT_FLAGS` 而不是 switch/case）
- `server.ts` 从 `commands.ts` 导入命令集，而不是内联声明
- SKILL.md 和 browse/SKILL.md 现在是生成文件（编辑 `.tmpl` 代替）

## 0.3.2 — 2026-03-13

### 修复
- Cookie 导入选择器现在返回 JSON 而不是 HTML — `jsonResponse()` 引用 `url` 超出范围，导致每次 API 调用崩溃
- `help` 命令正确路由（由于 META_COMMANDS 调度顺序而无法访问）
- 全局安装的陈旧服务器不再覆盖本地更改 — 从 `resolveServerScript()` 中移除了遗留的 `~/.claude/skills/gstack` 回退
- 崩溃日志路径引用从 `/tmp/` 更新到 `.gstack/`

### 新增
- **差异感知 QA 模式** — 特性分支上的 `/qa` 自动分析 `git diff`，识别受影响的页面/路由，检测本地主机上运行的应用，并仅测试更改的内容。无需 URL。
- **项目本地 browse 状态** — 状态文件、日志和所有服务器状态现在位于项目根目录内的 `.gstack/` 中（通过 `git rev-parse --show-toplevel` 检测）。不再有 `/tmp` 状态文件。
- **共享配置模块** (`browse/src/config.ts`) — 集中 CLI 和服务器的路径解析，消除重复的端口/状态逻辑
- **随机端口选择** — 服务器选择 10000-60000 之间的随机端口，而不是扫描 9400-9409。不再有 CONDUCTOR_PORT 魔法偏移。不再有工作区之间的端口冲突。
- **二进制版本跟踪** — 状态文件包含 `binaryVersion` SHA；CLI 在二进制重建时自动重启服务器
- **遗留 /tmp 清理** — CLI 扫描并删除旧的 `/tmp/browse-server*.json` 文件，在发送信号前验证 PID 所有权
- **Greptile 集成** — `/review` 和 `/ship` 获取并分类 Greptile 机器人评论；`/retro` 跟踪几周内的 Greptile 命中率
- **本地开发模式** — `bin/dev-setup` 从仓库符号链接技能用于就地开发；`bin/dev-teardown` 恢复全局安装
- `help` 命令 — 代理可以自发现所有命令和快照标志
- 版本感知 `find-browse` 带有 META 信号协议 — 检测陈旧二进制并提示代理更新
- `browse/dist/find-browse` 编译二进制，带有与 origin/main 的 git SHA 比较（4 小时缓存）
- `.version` 文件在构建时写入用于二进制版本跟踪
- Cookie 选择器的路由级测试（13 个测试）和 find-browse 版本检查（10 个测试）
- 配置解析测试（14 个测试），涵盖 git 根检测、BROWSE_STATE_FILE 覆盖、ensureStateDir、readVersionHash、resolveServerScript 和版本不匹配检测
- CLAUDE.md 中的浏览器交互指导 — 防止 Claude 使用 mcp__claude-in-chrome__* 工具
- CONTRIBUTING.md 带有快速入门、开发模式解释和在其他仓库中测试分支的说明

### 更改
- 状态文件位置：`.gstack/browse.json`（之前是 `/tmp/browse-server.json`）
- 日志文件位置：`.gstack/browse-{console,network,dialog}.log`（之前是 `/tmp/browse-*.log`）
- 原子状态文件写入：`.json.tmp` → 重命名（防止部分读取）
- CLI 将 `BROWSE_STATE_FILE` 传递给生成的服务器（服务器从它派生所有路径）
- SKILL.md 设置检查解析 META 信号并处理 `META:UPDATE_AVAILABLE`
- `/qa` SKILL.md 现在描述四种模式（差异感知、完整、快速、回归），在特性分支上差异感知为默认
- `jsonResponse`/`errorResponse` 使用选项对象防止位置参数混淆
- 构建脚本编译 `browse` 和 `find-browse` 二进制文件，清理 `.bun-build` 临时文件
- README 更新，包含 Greptile 设置说明、差异感知 QA 示例和修订的演示转录

### 移除
- `CONDUCTOR_PORT` 魔法偏移（`browse_port = CONDUCTOR_PORT - 45600`）
- 端口扫描范围 9400-9409
- 遗留回退到 `~/.claude/skills/gstack/browse/src/server.ts`
- `DEVELOPING_GSTACK.md`（重命名为 CONTRIBUTING.md）

## 0.3.1 — 2026-03-12

### 第 3.5 阶段：浏览器 cookie 导入

- `cookie-import-browser` 命令 — 从真实的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge）解密和导入 cookie
- 从 browse 服务器提供的交互式 cookie 选择器 web UI（深色主题，双面板布局，域搜索，导入/移除）
- 带有 `--domain` 标志的直接 CLI 导入，用于非交互式使用
- 用于 Claude Code 集成的 `/setup-browser-cookies` 技能
- 带有异步 10s 超时的 macOS Keychain 访问（无事件循环阻塞）
- 每浏览器 AES 密钥缓存（每个浏览器每个会话一个 Keychain 提示）
- DB 锁回退：将锁定的 cookie DB 复制到 /tmp 以安全读取
- 18 个单元测试，带有加密 cookie 夹具

## 0.3.0 — 2026-03-12

### 第 3 阶段：/qa 技能 — 系统性 QA 测试

- 新 `/qa` 技能，带有 6 阶段工作流（初始化、认证、定向、探索、文档、总结）
- 三种模式：完整（系统的，5-10 个问题），快速（30 秒冒烟测试），回归（与基线比较）
- 问题分类：7 个类别，4 个严重程度级别，每页探索清单
- 带有健康评分的结构化报告模板（0-100，跨 7 个类别加权）
- Next.js、Rails、WordPress 和 SPA 的框架检测指导
- `browse/bin/find-browse` — 使用 `git rev-parse --show-toplevel` 的 DRY 二进制发现

### 第 2 阶段：增强浏览器

- 对话框处理：自动接受/关闭，对话框缓冲区，提示文本支持
- 文件上传：`upload <sel> <file1> [file2...]`
- 元素状态检查：`is visible|hidden|enabled|disabled|checked|editable|focused <sel>`
- 带有 ref 标签覆盖的注释屏幕截图 (`snapshot -a`)
- 与之前快照的快照差异 (`snapshot -D`)
- 非 ARIA 可点击的光标交互式元素扫描 (`snapshot -C`)
- `wait --networkidle` / `--load` / `--domcontentloaded` 标志
- `console --errors` 过滤器（仅错误 + 警告）
- `cookie-import <json-file>` 带有从页面 URL 自动填充域
- 控制台/网络/对话框缓冲区的 CircularBuffer O(1) 环形缓冲区
- 带有 Bun.write() 的异步缓冲区刷新
- 带有 page.evaluate + 2s 超时的健康检查
- Playwright 错误包装 — AI 代理的可操作消息
- 上下文重建保留 cookie/存储/URL（用户代理修复）
- SKILL.md 重写为 QA 导向的剧本，带有 10 个工作流模式
- 166 个集成测试（之前约 63 个）

## 0.0.2 — 2026-03-12

- 修复项目本地 `/browse` 安装 — 编译的二进制现在从自己的目录解析 `server.ts`，而不是假设存在全局安装
- `setup` 重建陈旧的二进制（不仅仅是缺失的），如果构建失败则非零退出
- 修复 `chain` 命令吞噬写入命令的真实错误（例如导航超时报为 "Unknown meta command"）
- 修复 CLI 中服务器在同一命令上重复崩溃时的无限重启循环
- 将控制台/网络缓冲区限制在 50k 条目（环形缓冲区），而不是无限制增长
- 修复缓冲区达到 50k 上限后磁盘刷新静默停止
- 修复 setup 中的 `ln -snf` 以避免在升级时创建嵌套符号链接
- 使用 `git fetch && git reset --hard` 而不是 `git pull` 进行升级（处理强制推送）
- 简化安装：全局优先，可选项目副本（替换子模块方法）
- 重构 README：英雄，前后，演示转录，故障排除部分
- 六个技能（添加 `/retro`）

## 0.0.1 — 2026-03-11

初始版本。

- 五个技能：`/plan-ceo-review`、`/plan-eng-review`、`/review`、`/ship`、`/browse`
- 无头浏览器 CLI，带有 40+ 命令，基于 ref 的交互，持久 Chromium 守护进程
- 一键安装为 Claude Code 技能（子模块或全局克隆）
- `setup` 脚本用于二进制编译和技能符号链接