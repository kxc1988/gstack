# gstack 开发指南

## 命令

```bash
bun install          # 安装依赖
bun test             # 运行免费测试（browse + snapshot + skill 验证）
bun run test:evals   # 运行付费评估：LLM 评判 + E2E（基于差异，每次运行最高约 $4）
bun run test:evals:all  # 运行所有付费评估，无论差异如何
bun run test:e2e     # 仅运行 E2E 测试（基于差异，每次运行最高约 $3.85）
bun run test:e2e:all # 运行所有 E2E 测试，无论差异如何
bun run eval:select  # 基于当前差异显示将运行哪些测试
bun run dev <cmd>    # 在开发模式下运行 CLI，例如 bun run dev goto https://example.com
bun run build        # 生成文档 + 编译二进制文件
bun run gen:skill-docs  # 从模板重新生成 SKILL.md 文件
bun run skill:check  # 所有技能的健康仪表板
bun run dev:skill    # 监视模式：自动重新生成 + 变更时验证
bun run eval:list    # 列出 ~/.gstack-dev/evals/ 中的所有评估运行
bun run eval:compare # 比较两次评估运行（自动选择最近的两次）
bun run eval:summary # 汇总所有评估运行的统计数据
```

`test:evals` 需要 `ANTHROPIC_API_KEY`。E2E 测试通过 `--output-format stream-json --verbose` 实时流式传输进度（逐个工具）。结果会持久化到 `~/.gstack-dev/evals/`，并与之前的运行进行自动比较。

**基于差异的测试选择：** `test:evals` 和 `test:e2e` 会基于与基准分支的 `git diff` 自动选择测试。每个测试在 `test/helpers/touchfiles.ts` 中声明其文件依赖。对全局触摸文件（session-runner、eval-store、llm-judge、gen-skill-docs）的更改会触发所有测试。使用 `EVALS_ALL=1` 或 `:all` 脚本变体来强制运行所有测试。运行 `eval:select` 预览将运行哪些测试。

## 项目结构

```
gstack/
├── browse/          # 无头浏览器 CLI（Playwright）
│   ├── src/         # CLI + 服务器 + 命令
│   │   ├── commands.ts  # 命令注册表（单一事实来源）
│   │   └── snapshot.ts  # SNAPSHOT_FLAGS 元数据数组
│   ├── test/        # 集成测试 + 夹具
│   └── dist/        # 编译后的二进制文件
├── scripts/         # 构建 + DX 工具
│   ├── gen-skill-docs.ts  # 模板 → SKILL.md 生成器
│   ├── skill-check.ts     # 健康仪表板
│   └── dev-skill.ts       # 监视模式
├── test/            # 技能验证 + 评估测试
│   ├── helpers/     # skill-parser.ts, session-runner.ts, llm-judge.ts, eval-store.ts
│   ├── fixtures/    # 基准 JSON、植入错误的夹具、评估基线
│   ├── skill-validation.test.ts  # 第 1 层：静态验证（免费，<1s）
│   ├── gen-skill-docs.test.ts    # 第 1 层：生成器质量（免费，<1s）
│   ├── skill-llm-eval.test.ts   # 第 3 层：LLM 作为评判（~$0.15/运行）
│   └── skill-e2e.test.ts         # 第 2 层：通过 claude -p 进行 E2E（~$3.85/运行）
├── qa-only/         # /qa-only 技能（仅报告 QA，不修复）
├── plan-design-review/  # /plan-design-review 技能（仅报告设计审计）
├── design-review/    # /design-review 技能（设计审计 + 修复循环）
├── ship/            # 发布工作流技能
├── review/          # PR 审查技能
├── plan-ceo-review/ # /plan-ceo-review 技能
├── plan-eng-review/ # /plan-eng-review 技能
├── office-hours/    # /office-hours 技能（YC Office Hours — 创业诊断 + 构建者头脑风暴）
├── debug/           # /debug 技能（系统性根因调试）
├── retro/           # 回顾技能
├── document-release/ # /document-release 技能（发布后文档更新）
├── setup            # 一次性设置：构建二进制文件 + 符号链接技能
├── SKILL.md         # 从 SKILL.md.tmpl 生成（不要直接编辑）
├── SKILL.md.tmpl    # 模板：编辑此文件，运行 gen:skill-docs
└── package.json     # 用于 browse 的构建脚本
```

## SKILL.md 工作流程

SKILL.md 文件是从 `.tmpl` 模板**生成**的。要更新文档：

1. 编辑 `.tmpl` 文件（例如 `SKILL.md.tmpl` 或 `browse/SKILL.md.tmpl`）
2. 运行 `bun run gen:skill-docs`（或 `bun run build`，它会自动执行此操作）
3. 提交 `.tmpl` 和生成的 `.md` 文件

要添加新的 browse 命令：将其添加到 `browse/src/commands.ts` 并重建。
要添加快照标志：将其添加到 `browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS` 并重建。

## 编写 SKILL 模板

SKILL.md.tmpl 文件是**供 Claude 读取的提示模板**，不是 bash 脚本。每个 bash 代码块在单独的 shell 中运行 — 变量不会在块之间持久化。

规则：
- **使用自然语言进行逻辑和状态管理**。不要使用 shell 变量在代码块之间传递状态。相反，告诉 Claude 要记住什么，并在正文中引用它（例如，"步骤 0 中检测到的基准分支"）。
- **不要硬编码分支名称**。通过 `gh pr view` 或 `gh repo view` 动态检测 `main`/`master` 等。对于 PR 目标技能，使用 `{{BASE_BRANCH_DETECT}}`。在正文中使用 "基准分支"，在代码块占位符中使用 `<base>`。
- **保持 bash 块自包含**。每个代码块应独立工作。如果一个块需要来自前一步的上下文，请在上面的正文中重新陈述。
- **将条件表达为英语**。不要在 bash 中使用嵌套的 `if/elif/else`，而是编写编号的决策步骤："1. 如果 X，执行 Y。2. 否则，执行 Z。"

## 浏览器交互

当您需要与浏览器交互（QA、内部测试、cookie 设置）时，使用 `/browse` 技能或直接通过 `$B <command>` 运行 browse 二进制文件。**切勿使用** `mcp__claude-in-chrome__*` 工具 — 它们速度慢、不可靠，不是本项目使用的工具。

## 供应商符号链接感知

在开发 gstack 时，`.claude/skills/gstack` 可能是指向此工作目录的符号链接（被 git 忽略）。这意味着技能更改**立即生效** — 对于快速迭代非常有用，但在大型重构期间存在风险，因为半编写的技能可能会破坏同时使用 gstack 的其他 Claude Code 会话。

**每个会话检查一次：** 运行 `ls -la .claude/skills/gstack` 查看它是符号链接还是真实副本。如果它是指向您工作目录的符号链接，请注意：
- 模板更改 + `bun run gen:skill-docs` 会立即影响所有 gstack 调用
- 对 SKILL.md.tmpl 文件的破坏性更改可能会破坏并发的 gstack 会话
- 在大型重构期间，删除符号链接 (`rm .claude/skills/gstack`)，以便使用 `~/.claude/skills/gstack/` 中的全局安装

**对于计划审查：** 当审查修改技能模板或 gen-skill-docs 管道的计划时，考虑是否应在上线前单独测试这些更改（特别是如果用户在其他窗口中积极使用 gstack）。

## 提交风格

**始终对提交进行二分**。每个提交应该是一个单一的逻辑更改。当您进行了多项更改（例如，重命名 + 重写 + 新测试）时，在推送之前将它们拆分为单独的提交。每个提交都应该可以独立理解和回滚。

良好二分的例子：
- 重命名/移动与行为更改分开
- 测试基础设施（touchfiles、helpers）与测试实现分开
- 模板更改与生成文件的重新生成分开
- 机械重构与新功能分开

当用户说 "bisect commit" 或 "bisect and push" 时，将暂存/未暂存的更改拆分为逻辑提交并推送。

## CHANGELOG 风格

CHANGELOG.md 是**为用户**准备的，不是为贡献者。像产品发布说明一样编写：

- 以用户现在可以**做**以前不能做的事情开头。推销功能。
- 使用简单语言，不是实现细节。"您现在可以..." 而不是 "重构了..."
- **永远不要提及 TODOS.md、内部跟踪、评估基础设施或面向贡献者的细节**。这些对用户来说是不可见的，对他们来说毫无意义。
- 在底部的单独 "For contributors" 部分中放置贡献者/内部更改。
- 每个条目都应该让某人想 "哦，不错，我想试试那个。"
- 避免行话：说 "每个问题现在都告诉您您在哪个项目和分支中" 而不是 "通过前导解析器在技能模板中标准化 AskUserQuestion 格式。"

## AI 努力压缩

当估计或讨论工作量时，始终显示人类团队和 CC+gstack 时间：

| 任务类型 | 人类团队 | CC+gstack | 压缩比例 |
|-----------|-----------|-----------|-------------|
| 样板文件 / 脚手架 | 2 天 | 15 分钟 | ~100x |
| 测试编写 | 1 天 | 15 分钟 | ~50x |
| 功能实现 | 1 周 | 30 分钟 | ~30x |
| 错误修复 + 回归测试 | 4 小时 | 15 分钟 | ~20x |
| 架构 / 设计 | 2 天 | 4 小时 | ~5x |
| 研究 / 探索 | 1 天 | 3 小时 | ~3x |

完整性很便宜。当完整实现是 "湖泊"（可实现）而不是 "海洋"（多季度迁移）时，不要推荐捷径。有关完整理念，请参阅技能前导中的完整性原则。

## 本地计划

贡献者可以在 `~/.gstack-dev/plans/` 中存储长期愿景文档和设计文档。这些是仅本地的（不签入）。在审查 TODOS.md 时，检查 `plans/` 中可能已准备好提升为 TODO 或实施的候选者。

## E2E 评估失败责任协议

当 E2E 评估在 `/ship` 或任何其他工作流程中失败时，**在没有证明的情况下，永远不要声称 "与我们的更改无关"**。这些系统存在不可见的耦合 — 前导文本更改会影响代理行为，新助手会更改计时，重新生成的 SKILL.md 会改变提示上下文。

**在将失败归因于 "预先存在" 之前的要求：**
1. 在 main（或基准分支）上运行相同的评估，并显示它也在那里失败
2. 如果它在 main 上通过但在分支上失败 — 这是您的更改。追踪责任。
3. 如果您不能在 main 上运行，说 "未验证 — 可能相关也可能不相关" 并在 PR 正文中将其标记为风险

没有证据的 "预先存在" 是懒惰的说法。证明它或不要说它。

## 部署到活动技能

活动技能位于 `~/.claude/skills/gstack/`。进行更改后：

1. 推送您的分支
2. 在技能目录中获取并重置：`cd ~/.claude/skills/gstack && git fetch origin && git reset --hard origin/main`
3. 重建：`cd ~/.claude/skills/gstack && bun run build`

或者直接复制二进制文件：`cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`