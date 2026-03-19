# 为 gstack 做贡献

感谢您想要让 gstack 变得更好。无论您是修复技能提示中的拼写错误还是构建全新的工作流程，本指南都将帮助您快速上手。

## 快速开始

gstack 技能是 Claude Code 从 `skills/` 目录中发现的 Markdown 文件。通常它们位于 `~/.claude/skills/gstack/`（您的全局安装）。但是当您开发 gstack 本身时，您希望 Claude Code 使用*您的工作树中*的技能 — 这样编辑会立即生效，无需复制或部署任何内容。

这就是开发模式的作用。它将您的仓库符号链接到本地 `.claude/skills/` 目录，这样 Claude Code 就会直接从您的 checkout 读取技能。

```bash
git clone <repo> && cd gstack
bun install                    # 安装依赖
bin/dev-setup                  # 激活开发模式
```

现在编辑任何 `SKILL.md`，在 Claude Code 中调用它（例如 `/review`），并看到您的更改实时生效。开发完成后：

```bash
bin/dev-teardown               # 停用 — 回到您的全局安装
```

## 贡献者模式

贡献者模式将 gstack 转变为一个自我改进的工具。启用它后，Claude Code 会定期反思其 gstack 体验 — 在每个主要工作流程步骤结束时对其进行 0-10 评分。当某些事情不是 10 分时，它会思考原因并向 `~/.gstack/contributor-logs/` 提交一份报告，说明发生了什么、重现步骤以及如何改进。

```bash
~/.claude/skills/gstack/bin/gstack-config set gstack_contributor true
```

这些日志是为**您**准备的。当某件事让您足够烦恼以至于想要修复时，报告已经写好了。分叉 gstack，将您的分叉符号链接到您遇到问题的项目中，修复它，然后打开 PR。

### 贡献者工作流程

1. **正常使用 gstack** — 贡献者模式会自动反思并记录问题
2. **检查您的日志：** `ls ~/.gstack/contributor-logs/`
3. **分叉并克隆 gstack**（如果您还没有）
4. **将您的分叉符号链接到您遇到错误的项目中：**
   ```bash
   # 在您的核心项目中（那个让 gstack 惹恼您的项目）
   ln -sfn /path/to/your/gstack-fork .claude/skills/gstack
   cd .claude/skills/gstack && bun install && bun run build
   ```
5. **修复问题** — 您的更改在此项目中立即生效
6. **通过实际使用 gstack 进行测试** — 做那个惹恼您的事情，验证它是否已修复
7. **从您的分叉打开 PR**

这是贡献的最佳方式：在做实际工作时修复 gstack，在您真正感受到痛苦的项目中。

### 会话感知

当您同时打开 3 个以上的 gstack 会话时，每个问题都会告诉您哪个项目、哪个分支以及正在发生什么。不再盯着问题思考 "等等，这是哪个窗口？" 所有 15 个技能的格式都一致。

## 在 gstack 仓库内部处理 gstack

当您编辑 gstack 技能并想要通过在同一个仓库中实际使用 gstack 来测试它们时，`bin/dev-setup` 会连接起来。它创建 `.claude/skills/` 符号链接（被 git 忽略）指向您的工作树，这样 Claude Code 使用您的本地编辑而不是全局安装。

```
gstack/                          <- 您的工作树
├── .claude/skills/              <- 由 dev-setup 创建（被 git 忽略）
│   ├── gstack -> ../../         <- 指向仓库根目录的符号链接
│   ├── review -> gstack/review
│   ├── ship -> gstack/ship
│   └── ...                      <- 每个技能一个符号链接
├── review/
│   └── SKILL.md                 <- 编辑此文件，使用 /review 测试
├── ship/
│   └── SKILL.md
├── browse/
│   ├── src/                     <- TypeScript 源代码
│   └── dist/                    <- 编译后的二进制文件（被 git 忽略）
└── ...
```

## 日常工作流程

```bash
# 1. 进入开发模式
bin/dev-setup

# 2. 编辑技能
vim review/SKILL.md

# 3. 在 Claude Code 中测试 — 更改实时生效
#    > /review

# 4. 编辑 browse 源代码？重建二进制文件
bun run build

# 5. 当天完成？拆除
bin/dev-teardown
```

## 测试和评估

### 设置

```bash
# 1. 复制 .env.example 并添加您的 API 密钥
cp .env.example .env
# 编辑 .env → 设置 ANTHROPIC_API_KEY=sk-ant-...

# 2. 安装依赖（如果您还没有）
bun install
```

Bun 会自动加载 `.env` — 无需额外配置。Conductor 工作区会自动从主工作树继承 `.env`（见下面的 "Conductor 工作区"）。

### 测试层级

| 层级 | 命令 | 成本 | 测试内容 |
|------|---------|------|---------------|
| 1 — 静态 | `bun test` | 免费 | 命令验证、快照标志、SKILL.md 正确性、TODOS-format.md 引用、可观察性单元测试 |
| 2 — E2E | `bun run test:e2e` | ~$3.85 | 通过 `claude -p` 子进程的完整技能执行 |
| 3 — LLM 评估 | `bun run test:evals` | ~$0.15 独立 | 生成的 SKILL.md 文档的 LLM 作为评判评分 |
| 2+3 | `bun run test:evals` | ~$4 组合 | E2E + LLM 作为评判（运行两者） |

```bash
bun test                     # 仅第 1 层（每次提交运行，<5s）
bun run test:e2e             # 第 2 层：仅 E2E（需要 EVALS=1，不能在 Claude Code 内运行）
bun run test:evals           # 第 2 + 3 层组合（~$4/运行）
```

### 第 1 层：静态验证（免费）

使用 `bun test` 自动运行。不需要 API 密钥。

- **技能解析器测试** (`test/skill-parser.test.ts`) — 从 SKILL.md bash 代码块中提取每个 `$B` 命令并针对 `browse/src/commands.ts` 中的命令注册表进行验证。捕获拼写错误、已删除的命令和无效的快照标志。
- **技能验证测试** (`test/skill-validation.test.ts`) — 验证 SKILL.md 文件仅引用真实命令和标志，并且命令描述符合质量阈值。
- **生成器测试** (`test/gen-skill-docs.test.ts`) — 测试模板系统：验证占位符正确解析，输出包含标志的值提示（例如 `-d <N>` 而不仅仅是 `-d`），为关键命令提供丰富的描述（例如 `is` 列出有效状态，`press` 列出键示例）。

### 第 2 层：通过 `claude -p` 进行 E2E（~$3.85/运行）

以 `--output-format stream-json --verbose` 生成 `claude -p` 作为子进程，流式传输 NDJSON 以实时显示进度，并扫描浏览错误。这是最接近 "这个技能是否真的端到端工作？" 的测试。

```bash
# 必须从普通终端运行 — 不能嵌套在 Claude Code 或 Conductor 内部
EVALS=1 bun test test/skill-e2e.test.ts
```

- 由 `EVALS=1` 环境变量控制（防止意外的昂贵运行）
- 如果在 Claude Code 内部运行则自动跳过（`claude -p` 不能嵌套）
- API 连接预检查 — 在消耗预算之前快速失败于 ConnectionRefused
- 实时进度到 stderr：`[Ns] turn T tool #C: Name(...)`
- 保存完整的 NDJSON 转录和失败 JSON 用于调试
- 测试位于 `test/skill-e2e.test.ts`，运行器逻辑位于 `test/helpers/session-runner.ts`

### E2E 可观察性

当 E2E 测试运行时，它们会在 `~/.gstack-dev/` 中生成机器可读的工件：

| 工件 | 路径 | 目的 |
|----------|------|---------|
| 心跳 | `e2e-live.json` | 当前测试状态（每次工具调用更新） |
| 部分结果 | `evals/_partial-e2e.json` | 已完成的测试（在终止后仍然存在） |
| 进度日志 | `e2e-runs/{runId}/progress.log` | 仅追加文本日志 |
| NDJSON 转录 | `e2e-runs/{runId}/{test}.ndjson` | 每个测试的原始 `claude -p` 输出 |
| 失败 JSON | `e2e-runs/{runId}/{test}-failure.json` | 失败时的诊断数据 |

**实时仪表板：** 在第二个终端中运行 `bun run eval:watch` 以查看显示已完成测试、当前运行测试和成本的实时仪表板。使用 `--tail` 还可以显示 progress.log 的最后 10 行。

**评估历史工具：**

```bash
bun run eval:list            # 列出所有评估运行（轮次、持续时间、每次运行的成本）
bun run eval:compare         # 比较两次运行 — 显示每个测试的差异 + 摘要评论
bun run eval:summary         # 汇总统计数据 + 跨运行的每个测试效率平均值
```

**评估比较评论：** `eval:compare` 生成自然语言摘要部分，解释运行之间的变化 — 标记回归，注意改进，突出效率提升（更少的轮次、更快、更便宜），并产生整体摘要。这由 `eval-store.ts` 中的 `generateCommentary()` 驱动。

工件永远不会被清理 — 它们会在 `~/.gstack-dev/` 中积累，用于事后调试和趋势分析。

### 第 3 层：LLM 作为评判（~$0.15/运行）

使用 Claude Sonnet 从三个维度对生成的 SKILL.md 文档进行评分：

- **清晰度** — AI 代理能否无歧义地理解指令？
- **完整性** — 是否记录了所有命令、标志和使用模式？
- **可操作性** — 代理能否仅使用文档中的信息执行任务？

每个维度的评分范围为 1-5。阈值：每个维度的评分必须**≥ 4**。还有一个回归测试，将生成的文档与来自 `origin/main` 的手动维护基线进行比较 — 生成的评分必须等于或高于基线。

```bash
# 需要 .env 中的 ANTHROPIC_API_KEY — 包含在 bun run test:evals 中
```

- 使用 `claude-sonnet-4-6` 进行评分稳定性
- 测试位于 `test/skill-llm-eval.test.ts`
- 直接调用 Anthropic API（不是 `claude -p`），因此它可以从任何地方工作，包括 Claude Code 内部

### CI

GitHub Action（`.github/workflows/skill-docs.yml`）在每次推送和 PR 时运行 `bun run gen:skill-docs --dry-run`。如果生成的 SKILL.md 文件与已提交的文件不同，CI 会失败。这可以在合并之前捕获过时的文档。

测试直接针对 browse 二进制文件运行 — 它们不需要开发模式。

## 编辑 SKILL.md 文件

SKILL.md 文件是从 `.tmpl` 模板**生成**的。不要直接编辑 `.md` — 您的更改会在下次构建时被覆盖。

```bash
# 1. 编辑模板
vim SKILL.md.tmpl              # 或 browse/SKILL.md.tmpl

# 2. 重新生成
bun run gen:skill-docs

# 3. 检查健康状态
bun run skill:check

# 或使用监视模式 — 保存时自动重新生成
bun run dev:skill
```

有关模板编写最佳实践（自然语言而非 bash 风格、动态分支检测、`{{BASE_BRANCH_DETECT}}` 使用），请参阅 CLAUDE.md 的 "编写 SKILL 模板" 部分。

要添加 browse 命令，请将其添加到 `browse/src/commands.ts`。要添加快照标志，请将其添加到 `browse/src/snapshot.ts` 中的 `SNAPSHOT_FLAGS`。然后重建。

## Conductor 工作区

如果您使用 [Conductor](https://conductor.build) 并行运行多个 Claude Code 会话，`conductor.json` 会自动连接工作区生命周期：

| 钩子 | 脚本 | 作用 |
|------|--------|-------------|
| `setup` | `bin/dev-setup` | 从主工作树复制 `.env`，安装依赖，符号链接技能 |
| `archive` | `bin/dev-teardown` | 移除技能符号链接，清理 `.claude/` 目录 |

当 Conductor 创建新工作区时，`bin/dev-setup` 会自动运行。它检测主工作树（通过 `git worktree list`），复制您的 `.env` 以便 API 密钥 carry over，并设置开发模式 — 无需手动步骤。

**首次设置：** 在主仓库的 `.env` 中放入您的 `ANTHROPIC_API_KEY`（见 `.env.example`）。每个 Conductor 工作区都会自动继承它。

## 须知事项

- **SKILL.md 文件是生成的**。编辑 `.tmpl` 模板，不是 `.md`。运行 `bun run gen:skill-docs` 重新生成。
- **TODOS.md 是统一的待办事项列表**。按技能/组件组织，优先级为 P0-P4。`/ship` 自动检测已完成的项目。所有规划/审查/回顾技能都读取它以获取上下文。
- **Browse 源代码更改需要重建**。如果您修改 `browse/src/*.ts`，运行 `bun run build`。
- **开发模式会覆盖您的全局安装**。项目本地技能优先于 `~/.claude/skills/gstack`。`bin/dev-teardown` 会恢复全局安装。
- **Conductor 工作区是独立的**。每个工作区都是自己的 git worktree。`bin/dev-setup` 通过 `conductor.json` 自动运行。
- **`.env` 在工作树之间传播**。在主仓库中设置一次，所有 Conductor 工作区都会获得它。
- **`.claude/skills/` 被 git 忽略**。符号链接永远不会被提交。

## 在真实项目中测试您的更改

**这是开发 gstack 的推荐方式**。将您的 gstack checkout 符号链接到您实际使用它的项目中，这样您的更改在您做实际工作时会实时生效：

```bash
# 在您的核心项目中
ln -sfn /path/to/your/gstack-checkout .claude/skills/gstack
cd .claude/skills/gstack && bun install && bun run build
```

现在此项目中的每个 gstack 技能调用都使用您的工作树。编辑模板，运行 `bun run gen:skill-docs`，下一次 `/review` 或 `/qa` 调用会立即获取它。

**要回到稳定的全局安装**，只需删除符号链接：

```bash
rm .claude/skills/gstack
```

Claude Code 会自动回退到 `~/.claude/skills/gstack/`。

### 替代方案：将全局安装指向分支

如果您不想要每个项目的符号链接，您可以切换全局安装：

```bash
cd ~/.claude/skills/gstack
git fetch origin
git checkout origin/<branch>
bun install && bun run build
```

这会影响所有项目。要恢复：`git checkout main && git pull && bun run build`。

## 发布您的更改

当您对技能编辑满意时：

```bash
/ship
```

这会运行测试，审查差异，分类 Greptile 评论（带有 2 级升级），管理 TODOS.md，更新版本，并打开 PR。有关完整工作流程，请参阅 `ship/SKILL.md`。