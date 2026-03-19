---
name: document-release
version: 1.0.0
description: |
  发布后文档更新。读取所有项目文档，交叉引用差异，更新 README/ARCHITECTURE/CONTRIBUTING/CLAUDE.md 以匹配已发布的内容，
  优化 CHANGELOG 语气，清理 TODOS，并有选择地更新 VERSION。当被要求"更新文档"、"同步文档"或"发布后文档"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 序言（首先运行）

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
```

如果输出显示 `UPGRADE_AVAILABLE <old> <new>`：请阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并按照"内联升级流程"操作（如果配置了自动升级，则自动升级；否则，通过 AskUserQuestion 提供 4 个选项，如果用户拒绝，则写入 snooze 状态）。如果显示 `JUST_UPGRADED <from> <to>`：告诉用户"正在运行 gstack v{to}（刚刚更新！）"并继续。

如果 `LAKE_INTRO` 是 `no`：在继续之前，介绍完整性原则。
告诉用户："gstack 遵循 **煮沸湖泊** 原则 — 当 AI 使边际成本接近零时，总是做完整的事情。了解更多：https://garryslist.org/posts/boil-the-ocean"
然后询问是否在默认浏览器中打开这篇文章：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

只有在用户同意的情况下才运行 `open`。总是运行 `touch` 来标记为已查看。这只会发生一次。

## AskUserQuestion 格式

**每次调用 AskUserQuestion 时都必须遵循此结构：**
1. **重新接地：** 说明项目、当前分支（使用序言打印的 `_BRANCH` 值 — 而不是对话历史或 gitStatus 中的任何分支）以及当前计划/任务。（1-2 句话）
2. **简化：** 用一个聪明的 16 岁孩子都能理解的简单英语解释问题。不要使用原始函数名称，不要使用内部行话，不要使用实现细节。使用具体的例子和类比。说明它做什么，而不是它叫什么。
3. **推荐：** `RECOMMENDATION: Choose [X] because [one-line reason]` — 总是优先选择完整选项而不是快捷方式（参见完整性原则）。为每个选项包含 `Completeness: X/10`。校准：10 = 完整实现（所有边缘情况，全面覆盖），7 = 覆盖快乐路径但跳过一些边缘情况，3 = 推迟重要工作的快捷方式。如果两个选项都是 8+，选择更高的；如果一个是 ≤5，标记它。
4. **选项：** 带字母的选项：`A) ... B) ... C) ...` — 当选项涉及努力时，显示两个尺度：`(human: ~X / CC: ~Y)`

假设用户已经 20 分钟没有看这个窗口，并且没有打开代码。如果你需要阅读源代码来理解你自己的解释，那它太复杂了。

每个技能的说明可能会在此基线之上添加额外的格式规则。

## 完整性原则 — 煮沸湖泊

AI 辅助编码使完整性的边际成本接近零。当你提供选项时：

- 如果选项 A 是完整实现（完全对等，所有边缘情况，100% 覆盖）而选项 B 是节省适度努力的快捷方式 — **总是推荐 A**。在 CC+gstack 的情况下，80 行和 150 行之间的差异是无意义的。当"完整"只需要多几分钟时，"足够好"是错误的直觉。
- **湖泊 vs. 海洋：** "湖泊"是可以煮沸的 — 一个模块的 100% 测试覆盖，完整的功能实现，处理所有边缘情况，完整的错误路径。"海洋"则不是 — 从头重写整个系统，向你无法控制的依赖项添加功能，多季度的平台迁移。推荐煮沸湖泊。将海洋标记为超出范围。
- **当估计工作量时**，总是显示两个尺度：人类团队时间和 CC+gstack 时间。压缩比率因任务类型而异 — 使用以下参考：

| 任务类型 | 人类团队 | CC+gstack | 压缩比 |
|---------|---------|-----------|--------|
| 样板文件 / 脚手架 | 2 天 | 15 分钟 | ~100x |
| 测试编写 | 1 天 | 15 分钟 | ~50x |
| 功能实现 | 1 周 | 30 分钟 | ~30x |
| 错误修复 + 回归测试 | 4 小时 | 15 分钟 | ~20x |
| 架构 / 设计 | 2 天 | 4 小时 | ~5x |
| 研究 / 探索 | 1 天 | 3 小时 | ~3x |

- 此原则适用于测试覆盖率、错误处理、文档、边缘情况和功能完整性。不要为了"节省时间"而跳过最后 10% — 使用 AI，这 10% 只需要几秒钟。

**反模式 — 不要这样做：**
- 不好："选择 B — 它用更少的代码覆盖了 90% 的价值。"（如果 A 只多 70 行，选择 A。）
- 不好："我们可以跳过边缘情况处理来节省时间。"（使用 CC 处理边缘情况只需要几分钟。）
- 不好："让我们将测试覆盖推迟到后续 PR。"（测试是最容易煮沸的湖泊。）
- 不好：只引用人类团队的努力："这需要 2 周。"（应该说："2 周人类 / ~1 小时 CC。"）

## 贡献者模式

如果 `_CONTRIB` 是 `true`：你处于 **贡献者模式**。你是一个 gstack 用户，同时也帮助改进它。

**在每个主要工作流程步骤结束时**（不是在每个命令之后），反思你使用的 gstack 工具。将你的体验评为 0 到 10。如果不是 10，思考原因。如果有明显的、可操作的错误 OR 一个有见地的、有趣的事情，gstack 代码或技能 markdown 可以做得更好 — 提交一份现场报告。也许我们的贡献者会帮助我们变得更好！

**校准 — 这是标准：** 例如，`$B js "await fetch(...)"` 曾经因为 gstack 没有将表达式包装在异步上下文中而失败，错误为 `SyntaxError: await is only valid in async functions`。很小，但输入是合理的，gstack 应该处理它 — 这种事情值得提交。比这更不重要的事情，忽略。

**不值得提交：** 用户的应用程序错误，用户 URL 的网络错误，用户网站的身份验证失败，用户自己的 JS 逻辑错误。

**要提交：** 写入 `~/.gstack/contributor-logs/{slug}.md`，包含 **以下所有部分**（不要截断 — 包括直到日期/版本页脚的每个部分）：

```
# {标题}

嘿 gstack 团队 — 在使用 /{skill-name} 时遇到了这个问题：

**我试图做什么：** {用户/代理尝试的操作}
**实际发生了什么：** {实际发生的情况}
**我的评分：** {0-10} — {一句话说明为什么不是 10}

## 复现步骤
1. {步骤}

## 原始输出
```
{在此粘贴实际错误或意外输出}
```

## 什么会让这成为 10
{一句话：gstack 应该有什么不同的做法}

**日期：** {YYYY-MM-DD} | **版本：** {gstack 版本} | **技能：** /{skill}
```

Slug：小写，连字符，最多 60 个字符（例如 `browse-js-no-await`）。如果文件已存在，则跳过。每个会话最多 3 个报告。内联提交并继续 — 不要停止工作流程。告诉用户："已提交 gstack 现场报告：{标题}"

## 完成状态协议

完成技能工作流程时，使用以下之一报告状态：
- **DONE** — 所有步骤成功完成。为每个声明提供证据。
- **DONE_WITH_CONCERNS** — 已完成，但有用户应该知道的问题。列出每个问题。
- **BLOCKED** — 无法继续。说明什么在阻塞以及尝试了什么。
- **NEEDS_CONTEXT** — 缺少继续所需的信息。明确说明你需要什么。

### 升级

随时可以停止并说"这对我来说太难了"或"我对这个结果没有信心。"

糟糕的工作比没有工作更糟糕。你不会因为升级而受到惩罚。
- 如果你尝试了 3 次任务都没有成功，STOP 并升级。
- 如果你对安全敏感的更改不确定，STOP 并升级。
- 如果工作范围超出了你可以验证的范围，STOP 并升级。

升级格式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 句话]
ATTEMPTED: [你尝试了什么]
RECOMMENDATION: [用户接下来应该做什么]
```

## 步骤 0：检测基础分支

确定此 PR 针对哪个分支。在所有后续步骤中使用结果作为"基础分支"。

1. 检查是否已为此分支存在 PR：
   `gh pr view --json baseRefName -q .baseRefName`
   如果成功，使用打印的分支名称作为基础分支。

2. 如果不存在 PR（命令失败），检测仓库的默认分支：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 如果两个命令都失败，回退到 `main`。

打印检测到的基础分支名称。在每个后续的 `git diff`、`git log`、
`git fetch`、`git merge` 和 `gh pr create` 命令中，将指令中说"基础分支"的地方替换为检测到的分支名称。

---

# 文档发布：发布后文档更新

你正在运行 `/document-release` 工作流程。这在 **`/ship` 之后**（代码已提交，PR 存在或即将存在）但 **在 PR 合并之前** 运行。你的工作：确保项目中的每个文档文件都是准确、最新的，并以友好、面向用户的语气编写。

你大部分是自动化的。直接进行明显的事实更新。仅在有风险或主观决定时停止并询问。

**仅在以下情况停止：**
- 有风险/有问题的文档更改（叙述、哲学、安全性、删除、大型重写）
- VERSION 升级决策（如果尚未升级）
- 要添加的新 TODOS 项目
- 交叉文档矛盾是叙述性的（非事实性）

**永远不要在以下情况停止：**
- 明显来自差异的事实更正
- 向表格/列表添加项目
- 更新路径、计数、版本号
- 修复过时的交叉引用
- CHANGELOG 语气优化（轻微措辞调整）
- 标记 TODOS 完成
- 交叉文档事实不一致（例如，版本号不匹配）

**永远不要：**
- 覆盖、替换或重新生成 CHANGELOG 条目 — 仅优化措辞，保留所有内容
- 未经询问就升级 VERSION — 版本更改始终使用 AskUserQuestion
- 对 CHANGELOG.md 使用 `Write` 工具 — 始终使用 `Edit` 并精确匹配 `old_string`

---

## 步骤 1：飞行前和差异分析

1. 检查当前分支。如果在基础分支上，**中止**："你在基础分支上。从功能分支运行。"

2. 收集关于更改内容的上下文：

```bash
git diff <base>...HEAD --stat
```

```bash
git log <base>..HEAD --oneline
```

```bash
git diff <base>...HEAD --name-only
```

3. 发现仓库中的所有文档文件：

```bash
find . -maxdepth 2 -name "*.md" -not -path "./.git/*" -not -path "./node_modules/*" -not -path "./.gstack/*" -not -path "./.context/*" | sort
```

4. 将更改分类为与文档相关的类别：
   - **新功能** — 新文件、新命令、新技能、新功能
   - **行为更改** — 修改的服务、更新的 API、配置更改
   - **已移除功能** — 删除的文件、移除的命令
   - **基础设施** — 构建系统、测试基础设施、CI

5. 输出简短摘要："分析 N 个文件在 M 个提交中更改。发现 K 个文档文件需要审查。"

---

## 步骤 2：逐文件文档审计

阅读每个文档文件并将其与差异交叉引用。使用这些通用启发式方法
（适应你所在的任何项目 — 这些不是 gstack 特定的）：

**README.md：**
- 它是否描述了差异中可见的所有功能和能力？
- 安装/设置说明是否与更改一致？
- 示例、演示和使用说明是否仍然有效？
- 故障排除步骤是否仍然准确？

**ARCHITECTURE.md：**
- ASCII 图表和组件描述是否与当前代码匹配？
- 设计决策和"为什么"解释是否仍然准确？
- 保守处理 — 只更新被差异明确反驳的内容。架构文档
  描述不太可能频繁更改的内容。

**CONTRIBUTING.md — 新贡献者冒烟测试：**
- 像全新贡献者一样浏览设置说明。
- 列出的命令是否准确？每个步骤都会成功吗？
- 测试层描述是否与当前测试基础设施匹配？
- 工作流程描述（开发设置、贡献者模式等）是否最新？
- 标记任何会失败或混淆首次贡献者的内容。

**CLAUDE.md / 项目说明：**
- 项目结构部分是否与实际文件树匹配？
- 列出的命令和脚本是否准确？
- 构建/测试说明是否与 package.json（或等效文件）中的内容匹配？

**任何其他 .md 文件：**
- 阅读文件，确定其目的和受众。
- 与差异交叉引用，检查是否与文件所述内容相矛盾。

对于每个文件，将需要的更新分类为：

- **自动更新** — 差异明确保证的事实更正：向表中添加项目，
  更新文件路径，修复计数，更新项目结构树。
- **询问用户** — 叙述性更改、部分删除、安全模型更改、大型重写
  （一个部分超过 ~10 行）、模棱两可的相关性、添加全新部分。

---

## 步骤 3：应用自动更新

使用 Edit 工具直接进行所有明确的事实更新。

对于每个修改的文件，输出一行摘要，描述 **具体更改了什么** — 不是
只是"更新了 README.md"，而是"README.md：将 /new-skill 添加到技能表，将技能计数从 9 更新到 10"。

**永远不要自动更新：**
- README 介绍或项目定位
- ARCHITECTURE 理念或设计原理
- 安全模型描述
- 不要从任何文档中删除整个部分

---

## 步骤 4：询问关于有风险/有问题的更改

对于步骤 2 中识别的每个有风险或有问题的更新，使用 AskUserQuestion，包含：
- 上下文：项目名称、分支、哪个文档文件、我们正在审查什么
- 具体的文档决策
- `RECOMMENDATION: Choose [X] because [one-line reason]`
- 选项包括 C) 跳过 — 保持原样

在每个答案后立即应用批准的更改。

---

## 步骤 5：CHANGELOG 语气优化

**关键 — 永远不要覆盖 CHANGELOG 条目。**

此步骤优化语气。它不会重写、替换或重新生成 CHANGELOG 内容。

曾经发生过一个真实事件，代理在应该保留现有 CHANGELOG 条目的情况下替换了它们。此技能绝对不能这样做。

**规则：**
1. 首先阅读整个 CHANGELOG.md。了解已经存在的内容。
2. 仅修改现有条目中的措辞。永远不要删除、重新排序或替换条目。
3. 永远不要从头重新生成 CHANGELOG 条目。条目是由 `/ship` 从
   实际差异和提交历史中写入的。它是真相的来源。你正在优化散文，不是
   重写历史。
4. 如果条目看起来错误或不完整，使用 AskUserQuestion — 不要默默地修复它。
5. 使用 Edit 工具并精确匹配 `old_string` — 永远不要使用 Write 覆盖 CHANGELOG.md。

**如果此分支中未修改 CHANGELOG：** 跳过此步骤。

**如果此分支中修改了 CHANGELOG**，审查条目的语气：

- **销售测试：** 阅读每个项目符号的用户是否会想"哦，不错，我想试试那个"？如果不是，
  重写措辞（不是内容）。
- 以用户现在可以 **做** 什么开头 — 不是实现细节。
- "你现在可以..." 而不是 "重构了..."
- 标记并重写任何读起来像提交消息的条目。
- 内部/贡献者更改属于单独的"### For contributors"子部分。
- 自动修复轻微的语气调整。如果重写会改变含义，使用 AskUserQuestion。

---

## 步骤 6：交叉文档一致性和可发现性检查

在单独审计每个文件后，进行交叉文档一致性检查：

1. README 的功能/能力列表是否与 CLAUDE.md（或项目说明）描述的内容匹配？
2. ARCHITECTURE 的组件列表是否与 CONTRIBUTING 的项目结构描述匹配？
3. CHANGELOG 的最新版本是否与 VERSION 文件匹配？
4. **可发现性：** 每个文档文件是否可从 README.md 或 CLAUDE.md 访问？如果
   ARCHITECTURE.md 存在但 README 和 CLAUDE.md 都没有链接到它，标记它。每个文档
   应该可从两个入口点文件之一访问。
5. 标记文档之间的任何矛盾。自动修复明确的事实不一致（例如，
   版本不匹配）。对于叙述性矛盾，使用 AskUserQuestion。

---

## 步骤 7：TODOS.md 清理

这是对 `/ship` 的步骤 5.5 的补充。阅读 `review/TODOS-format.md`（如果
可用）以获取规范的 TODO 项目格式。

如果 TODOS.md 不存在，跳过此步骤。

1. **尚未标记的已完成项目：** 将差异与未完成的 TODO 项目交叉引用。如果一个
   TODO 显然由该分支中的更改完成，将其移动到已完成部分，
   并标记 `**Completed:** vX.Y.Z.W (YYYY-MM-DD)`。保守处理 — 只标记差异中有明确
   证据的项目。

2. **需要描述更新的项目：** 如果 TODO 引用了已显著更改的文件或组件，其描述可能已过时。使用 AskUserQuestion 确认该 TODO 是否应更新、完成或保持原样。

3. **新的延期工作：** 检查差异中的 `TODO`、`FIXME`、`HACK` 和 `XXX` 注释。对于
   每个代表有意义的延期工作（不是微不足道的内联注释），使用
   AskUserQuestion 询问是否应将其捕获在 TODOS.md 中。

---

## 步骤 8：VERSION 升级问题

**关键 — 永远不要在未经询问的情况下升级 VERSION。**

1. **如果 VERSION 不存在：** 静默跳过。

2. 检查此分支上是否已修改 VERSION：

```bash
git diff <base>...HEAD -- VERSION
```

3. **如果 VERSION 尚未升级：** 使用 AskUserQuestion：
   - RECOMMENDATION: 选择 C（跳过），因为仅文档更改很少需要版本升级
   - A) 升级 PATCH（X.Y.Z+1）— 如果文档更改与代码更改一起发布
   - B) 升级 MINOR（X.Y+1.0）— 如果这是一个重要的独立发布
   - C) 跳过 — 不需要版本升级

4. **如果 VERSION 已经升级：** 不要静默跳过。相反，检查升级是否仍然覆盖此分支上更改的全部范围：

   a. 阅读当前 VERSION 的 CHANGELOG 条目。它描述了哪些功能？
   b. 阅读完整差异（`git diff <base>...HEAD --stat` 和 `git diff <base>...HEAD --name-only`）。
      是否有未在当前版本的 CHANGELOG 条目中提及的重大更改（新功能、新技能、新命令、重大重构）？
   c. **如果 CHANGELOG 条目涵盖了所有内容：** 跳过 — 输出 "VERSION: Already bumped to
      vX.Y.Z, covers all changes."
   d. **如果有未涵盖的重大更改：** 使用 AskUserQuestion 解释当前版本涵盖的内容与新增内容，然后询问：
      - RECOMMENDATION: 选择 A，因为新更改值得拥有自己的版本
      - A) 升级到下一个补丁（X.Y.Z+1）— 为新更改提供自己的版本
      - B) 保持当前版本 — 将新更改添加到现有 CHANGELOG 条目
      - C) 跳过 — 保持版本不变，稍后处理

   关键洞察：为"功能 A"设置的 VERSION 升级不应静默吸收"功能 B"，
   如果功能 B 足够重要，值得拥有自己的版本条目。

---

## 步骤 9：提交和输出

**先检查是否为空：** 运行 `git status`（永远不要使用 `-uall`）。如果任何先前步骤都未修改文档文件，输出"所有文档都是最新的。"并退出，不提交。

**提交：**

1. 按名称暂存修改的文档文件（永远不要 `git add -A` 或 `git add .`）。
2. 创建单个提交：

```bash
git commit -m "$(cat <<'EOF'
docs: update project documentation for vX.Y.Z.W

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

3. 推送到当前分支：

```bash
git push
```

**PR 正文更新（幂等，防竞争）：**

1. 将现有 PR 正文读取到 PID 唯一的临时文件：

```bash
gh pr view --json body -q .body > /tmp/gstack-pr-body-$$.md
```

2. 如果临时文件已包含 `## Documentation` 部分，用更新的内容替换该部分。如果不包含，在末尾追加 `## Documentation` 部分。

3. Documentation 部分应包含 **文档差异预览** — 对于每个修改的文件，
   描述具体更改了什么（例如，"README.md：将 /document-release 添加到技能
   表，将技能计数从 9 更新到 10"）。

4. 将更新的正文写回：

```bash
gh pr edit --body-file /tmp/gstack-pr-body-$$.md
```

5. 清理临时文件：

```bash
rm -f /tmp/gstack-pr-body-$$.md
```

6. 如果 `gh pr view` 失败（不存在 PR）：跳过，消息为 "No PR found — skipping body update."
7. 如果 `gh pr edit` 失败：警告 "Could not update PR body — documentation changes are in the
   commit." 并继续。

**结构化文档健康摘要（最终输出）：**

输出一个可扫描的摘要，显示每个文档文件的状态：

```
Documentation health:
  README.md       [status] ([details])
  ARCHITECTURE.md [status] ([details])
  CONTRIBUTING.md [status] ([details])
  CHANGELOG.md    [status] ([details])
  TODOS.md        [status] ([details])
  VERSION         [status] ([details])
```

其中 status 是以下之一：
- Updated — 描述更改内容
- Current — 不需要更改
- Voice polished — 措辞已调整
- Not bumped — 用户选择跳过
- Already bumped — 版本由 /ship 设置
- Skipped — 文件不存在

---

## 重要规则

- **编辑前阅读。** 修改文件前始终阅读其完整内容。
- **永远不要覆盖 CHANGELOG。** 仅优化措辞。永远不要删除、替换或重新生成条目。
- **永远不要静默升级 VERSION。** 始终询问。即使已经升级，也要检查它是否覆盖了更改的全部范围。
- **明确说明更改内容。** 每次编辑都要有一行摘要。
- **通用启发式方法，非项目特定。** 审计检查适用于任何仓库。
- **可发现性很重要。** 每个文档文件都应可从 README 或 CLAUDE.md 访问。
- **语气：友好，面向用户，不晦涩。** 写作时就像向一个聪明但未见过代码的人解释。