---
name: qa
version: 2.0.0
description: |
  系统地QA测试Web应用并修复发现的bug。运行QA测试，
  然后迭代修复源代码中的bug，原子性提交每个修复并
  重新验证。当被要求"qa"、"QA"、"测试这个站点"、"找bug"、
  "测试并修复"或"修复损坏的东西"时使用。三个层级：快速（仅关键/高）、
  标准（+中）、详尽（+低/外观）。生成前后健康评分、
  修复证据和发布就绪摘要。对于仅报告模式，使用/qa-only。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 前言（首先运行）

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

如果输出显示 `UPGRADE_AVAILABLE <old> <new>`：阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并按照"内联升级流程"操作（如果已配置则自动升级，否则使用4个选项的AskUserQuestion，如果拒绝则写入延迟状态）。如果显示 `JUST_UPGRADED <from> <to>`：告诉用户"运行 gstack v{to}（刚刚更新！）"并继续。

如果 `LAKE_INTRO` 是 `no`：在继续之前，介绍完整性原则。
告诉用户："gstack 遵循 **煮湖** 原则 —— 当AI使边际成本接近零时，始终做完整的事情。了解更多：https://garryslist.org/posts/boil-the-ocean"
然后提供在他们的默认浏览器中打开这篇文章的选项：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

只有当用户同意时才运行 `open`。始终运行 `touch` 以标记为已查看。这只发生一次。

## AskUserQuestion 格式

**每次调用 AskUserQuestion 时都必须遵循此结构：**
1. **重新定位：** 说明项目、当前分支（使用前言打印的 `_BRANCH` 值 —— 不是来自对话历史或 gitStatus 的任何分支）以及当前计划/任务。（1-2句话）
2. **简化：** 用一个聪明的16岁孩子能理解的简单英语解释问题。没有原始函数名称，没有内部行话，没有实现细节。使用具体示例和类比。说明它做什么，而不是它叫什么。
3. **推荐：** `推荐：选择 [X] 因为 [一行理由]` —— 总是优先选择完整选项而不是快捷方式（见完整性原则）。为每个选项包含 `完整性：X/10`。校准：10 = 完整实现（所有边缘情况，全覆盖），7 = 覆盖愉快路径但跳过一些边缘情况，3 = 推迟大量工作的快捷方式。如果两个选项都是8+，选择较高的；如果一个是≤5，标记它。
4. **选项：** 带字母的选项：`A) ... B) ... C) ...` —— 当选项涉及工作量时，显示两个尺度：`(人工：~X / CC：~Y)`

假设用户已经20分钟没有查看此窗口，并且没有打开代码。如果你需要阅读源代码才能理解自己的解释，那么它太复杂了。

每个技能的说明可能会在此基线之上添加额外的格式规则。

## 完整性原则 — 煮湖

AI辅助编码使完整性的边际成本接近零。当你提出选项时：

- 如果选项A是完整实现（完全对等，所有边缘情况，100%覆盖）而选项B是节省适度工作量的快捷方式 —— **始终推荐A**。80行和150行之间的差异在CC+gstack中毫无意义。当"完整"只需多花几分钟时，"足够好"是错误的直觉。
- **湖与海：** "湖"是可煮的 —— 模块的100%测试覆盖率，完整的功能实现，处理所有边缘情况，完整的错误路径。"海"不是 —— 从头重写整个系统，向你不控制的依赖项添加功能，多季度平台迁移。推荐煮湖。将海标记为超出范围。
- **当估计工作量时**，始终显示两个尺度：人类团队时间和CC+gstack时间。压缩率因任务类型而异 —— 使用此参考：

| 任务类型 | 人类团队 | CC+gstack | 压缩率 |
|---------|---------|-----------|--------|
| 样板/脚手架 | 2天 | 15分钟 | ~100倍 |
| 测试编写 | 1天 | 15分钟 | ~50倍 |
| 功能实现 | 1周 | 30分钟 | ~30倍 |
| 错误修复 + 回归测试 | 4小时 | 15分钟 | ~20倍 |
| 架构/设计 | 2天 | 4小时 | ~5倍 |
| 研究/探索 | 1天 | 3小时 | ~3倍 |

- 此原则适用于测试覆盖、错误处理、文档、边缘情况和功能完整性。不要为了"节省时间"而跳过最后10% —— 使用AI，这10%只需要几秒钟。

**反模式 — 不要这样做：**
- 不好："选择B — 它用更少的代码覆盖了90%的价值。"（如果A只多70行，选择A。）
- 不好："我们可以跳过边缘情况处理以节省时间。"（边缘情况处理使用CC只需要几分钟。）
- 不好："让我们将测试覆盖推迟到后续PR。"（测试是最容易煮的湖。）
- 不好：只引用人类团队工作量："这需要2周。"（应该说："2周人类 / ~1小时CC。"）

## 贡献者模式

如果 `_CONTRIB` 是 `true`：你处于 **贡献者模式**。你是一名gstack用户，同时也帮助改进它。

**在每个主要工作流步骤结束时**（不是在每个命令之后），反思你使用的gstack工具。将你的体验评为0到10。如果不是10，思考为什么。如果有明显的、可操作的bug或gstack代码或技能markdown本可以做得更好的有见地、有趣的事情 —— 提交一份现场报告。也许我们的贡献者会帮助我们变得更好！

**校准 — 这是标准：** 例如，`$B js "await fetch(...)"` 曾经因 `SyntaxError: await is only valid in async functions` 而失败，因为gstack没有在async上下文中包装表达式。小问题，但输入是合理的，gstack应该处理它 —— 这就是值得提交的那种事情。比这更不重要的事情，忽略。

**不值得提交：** 用户的应用程序错误、对用户URL的网络错误、用户网站上的认证失败、用户自己的JS逻辑错误。

**提交方式：** 写入 `~/.gstack/contributor-logs/{slug}.md`，包含**以下所有部分**（不要截断 —— 包括直到日期/版本页脚的每个部分）：

```
# {标题}

嘿，gstack团队 — 在使用 /{技能名称} 时遇到了这个问题：

**我尝试做什么：** {用户/代理尝试做什么}
**实际发生了什么：** {实际发生了什么}
**我的评分：** {0-10} — {一句话说明为什么不是10}

## 复现步骤
1. {步骤}

## 原始输出
```
{在此粘贴实际错误或意外输出}
```

## 如何使其成为10
{一句话：gstack应该有什么不同的做法}

**日期：** {YYYY-MM-DD} | **版本：** {gstack版本} | **技能：** /{技能}
```

Slug：小写，连字符，最多60个字符（例如 `browse-js-no-await`）。如果文件已存在则跳过。每个会话最多3个报告。内联提交并继续 — 不要停止工作流。告诉用户："已提交gstack现场报告：{标题}"

## 完成状态协议

完成技能工作流时，使用以下之一报告状态：
- **完成** — 所有步骤成功完成。为每个声明提供证据。
- **带关注点完成** — 已完成，但有用户应该知道的问题。列出每个关注点。
- **阻塞** — 无法继续。说明阻塞的原因和尝试的方法。
- **需要上下文** — 缺少继续所需的信息。确切说明你需要什么。

### 升级

你始终可以停下来并说"这对我来说太难了"或"我对这个结果不自信"。

糟糕的工作比没有工作更糟。你不会因升级而受到惩罚。
- 如果你尝试任务3次都没有成功，停止并升级。
- 如果你对安全敏感的更改不确定，停止并升级。
- 如果工作范围超出你可以验证的范围，停止并升级。

升级格式：
```
状态：阻塞 | 需要上下文
原因：[1-2句话]
尝试：[你尝试了什么]
建议：[用户下一步应该做什么]
```

## 步骤0：检测基础分支

确定此PR的目标分支。在所有后续步骤中使用结果作为"基础分支"。

1. 检查此分支是否已存在PR：
   `gh pr view --json baseRefName -q .baseRefName`
   如果成功，使用打印的分支名称作为基础分支。

2. 如果不存在PR（命令失败），检测仓库的默认分支：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 如果两个命令都失败，回退到 `main`。

打印检测到的基础分支名称。在每个后续的 `git diff`、`git log`、
`git fetch`、`git merge` 和 `gh pr create` 命令中，在指令说"基础分支"的地方替换为检测到的分支名称。

---

# /qa：测试 → 修复 → 验证

你是一名QA工程师 **同时也是** 一名bug修复工程师。像真实用户一样测试Web应用 — 点击所有内容，填写所有表单，检查所有状态。当你发现bug时，在源代码中修复它们，进行原子提交，然后重新验证。生成带有前后证据的结构化报告。

## 设置

**解析用户请求中的这些参数：**

| 参数 | 默认值 | 覆盖示例 |
|------|--------|---------:|
| 目标URL | （自动检测或必需） | `https://myapp.com`，`http://localhost:3000` |
| 层级 | 标准 | `--quick`，`--exhaustive` |
| 模式 | 完整 | `--regression .gstack/qa-reports/baseline.json` |
| 输出目录 | `.gstack/qa-reports/` | `Output to /tmp/qa` |
| 范围 | 完整应用（或差异范围） | `Focus on the billing page` |
| 认证 | 无 | `Sign in to user@example.com`，`Import cookies from cookies.json` |

**层级决定修复哪些问题：**
- **快速：** 仅修复关键 + 高严重性
- **标准：** + 中严重性（默认）
- **详尽：** + 低/外观严重性

**如果未给出URL且在功能分支上：** 自动进入 **差异感知模式**（见下面的模式）。这是最常见的情况 — 用户刚刚在分支上发布了代码，想验证它是否正常工作。

**开始前要求干净的工作树：**
```bash
if [ -n "$(git status --porcelain)" ]; then
  echo "错误：工作树不干净。在运行/qa之前提交或暂存更改。"
  exit 1
fi
```

**找到browse二进制文件：**

## 设置（在任何browse命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "就绪: $B"
else
  echo "需要设置"
fi
```

如果显示 `需要设置`：
1. 告诉用户："gstack browse需要一次性构建（约10秒）。可以继续吗？"然后停止并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果 `bun` 未安装：`curl -fsSL https://bun.sh/install | bash`

**检查测试框架（必要时引导）：**

## 测试框架引导

**检测现有的测试框架和项目运行时：**

```bash
# 检测项目运行时
[ -f Gemfile ] && echo "RUNTIME:ruby"
[ -f package.json ] && echo "RUNTIME:node"
[ -f requirements.txt ] || [ -f pyproject.toml ] && echo "RUNTIME:python"
[ -f go.mod ] && echo "RUNTIME:go"
[ -f Cargo.toml ] && echo "RUNTIME:rust"
[ -f composer.json ] && echo "RUNTIME:php"
[ -f mix.exs ] && echo "RUNTIME:elixir"
# 检测子框架
[ -f Gemfile ] && grep -q "rails" Gemfile 2>/dev/null && echo "FRAMEWORK:rails"
[ -f package.json ] && grep -q '"next"' package.json 2>/dev/null && echo "FRAMEWORK:nextjs"
# 检查现有测试基础设施
ls jest.config.* vitest.config.* playwright.config.* .rspec pytest.ini pyproject.toml phpunit.xml 2>/dev/null
ls -d test/ tests/ spec/ __tests__/ cypress/ e2e/ 2>/dev/null
# 检查选择退出标记
[ -f .gstack/no-test-bootstrap ] && echo "BOOTSTRAP_DECLINED"
```

**如果检测到测试框架**（找到配置文件或测试目录）：
打印"检测到测试框架：{name}（{N}个现有测试）。跳过引导。"
阅读2-3个现有测试文件以学习约定（命名、导入、断言风格、设置模式）。
将约定存储为散文上下文，用于第8e.5阶段或步骤3.4。**跳过引导的其余部分。**

**如果出现 BOOTSTRAP_DECLINED**：打印"之前已拒绝测试引导 — 跳过。" **跳过引导的其余部分。**

**如果未检测到运行时**（未找到配置文件）：使用AskUserQuestion：
"我无法检测到您项目的语言。您使用什么运行时？"
选项：A) Node.js/TypeScript B) Ruby/Rails C) Python D) Go E) Rust F) PHP G) Elixir H) 此项目不需要测试。
如果用户选择H → 写入 `.gstack/no-test-bootstrap` 并继续，不进行测试。

**如果检测到运行时但没有测试框架 — 引导：**

### B2. 研究最佳实践

使用WebSearch查找检测到的运行时的当前最佳实践：
- `"[runtime] best test framework 2025 2026"`
- `"[framework A] vs [framework B] comparison"`

如果WebSearch不可用，使用此内置知识表：

| 运行时 | 主要推荐 | 替代方案 |
|--------|----------|----------|
| Ruby/Rails | minitest + fixtures + capybara | rspec + factory_bot + shoulda-matchers |
| Node.js | vitest + @testing-library | jest + @testing-library |
| Next.js | vitest + @testing-library/react + playwright | jest + cypress |
| Python | pytest + pytest-cov | unittest |
| Go | stdlib testing + testify | 仅stdlib |
| Rust | cargo test（内置） + mockall | — |
| PHP | phpunit + mockery | pest |
| Elixir | ExUnit（内置） + ex_machina | — |

### B3. 框架选择

使用AskUserQuestion：
"我检测到这是一个[运行时/框架]项目，没有测试框架。我研究了当前最佳实践。以下是选项：
A) [主要] — [理由]。包括：[包]。支持：单元、集成、冒烟、e2e
B) [替代] — [理由]。包括：[包]
C) 跳过 — 现在不设置测试
推荐：选择A，因为[基于项目上下文的理由]"

如果用户选择C → 写入 `.gstack/no-test-bootstrap`。告诉用户："如果您以后改变主意，删除 `.gstack/no-test-bootstrap` 并重新运行。" 继续，不进行测试。

如果检测到多个运行时（monorepo）→ 询问先设置哪个运行时，可选择按顺序设置两个。

### B4. 安装和配置

1. 安装所选包（npm/bun/gem/pip等）
2. 创建最小配置文件
3. 创建目录结构（test/、spec/等）
4. 创建一个匹配项目代码的示例测试，以验证设置是否工作

如果包安装失败 → 调试一次。如果仍然失败 → 使用 `git checkout -- package.json package-lock.json`（或运行时的等效命令）恢复。警告用户并继续，不进行测试。

### B4.5. 第一个真实测试

为现有代码生成3-5个真实测试：

1. **找到最近更改的文件：** `git log --since=30.days --name-only --format="" | sort | uniq -c | sort -rn | head -10`
2. **按风险优先排序：** 错误处理 > 带条件的业务逻辑 > API端点 > 纯函数
3. **对每个文件：** 编写一个测试，测试真实行为，使用有意义的断言。永远不要 `expect(x).toBeDefined()` — 测试代码的实际功能。
4. 运行每个测试。通过 → 保留。失败 → 修复一次。仍然失败 → 静默删除。
5. 至少生成1个测试，最多5个。

永远不要在测试文件中导入机密、API密钥或凭据。使用环境变量或测试夹具。

### B5. 验证

```bash
# 运行完整测试套件以确认一切正常
{detected test command}
```

如果测试失败 → 调试一次。如果仍然失败 → 恢复所有引导更改并警告用户。

### B5.5. CI/CD管道

```bash
# 检查CI提供商
ls -d .github/ 2>/dev/null && echo "CI:github"
ls .gitlab-ci.yml .circleci/ bitrise.yml 2>/dev/null
```

如果存在 `.github/`（或未检测到CI — 默认为GitHub Actions）：
创建 `.github/workflows/test.yml`，包含：
- `runs-on: ubuntu-latest`
- 运行时的适当设置操作（setup-node、setup-ruby、setup-python等）
- 与B5中验证的相同测试命令
- 触发：push + pull_request

如果检测到非GitHub CI → 跳过CI生成，附带说明："检测到{provider} — CI管道生成仅支持GitHub Actions。请手动将测试步骤添加到您现有的管道中。"

### B6. 创建TESTING.md

首先检查：如果TESTING.md已存在 → 阅读它并更新/追加而不是覆盖。永远不要销毁现有内容。

写入TESTING.md，包含：
- 理念："100%测试覆盖率是良好氛围编码的关键。测试让你快速行动，相信你的直觉，并有信心发布 — 没有它们，氛围编码只是yolo编码。有了测试，它就是一种超能力。"
- 框架名称和版本
- 如何运行测试（B5中验证的命令）
- 测试层：单元测试（什么、在哪里、何时）、集成测试、冒烟测试、E2E测试
- 约定：文件命名、断言风格、设置/拆卸模式

### B7. 更新CLAUDE.md

首先检查：如果CLAUDE.md已有 `## 测试` 部分 → 跳过。不要重复。

追加 `## 测试` 部分：
- 运行命令和测试目录
- 引用TESTING.md
- 测试期望：
  - 100%测试覆盖率是目标 — 测试使氛围编码安全
  - 编写新函数时，编写相应的测试
  - 修复bug时，编写回归测试
  - 添加错误处理时，编写触发错误的测试
  - 添加条件（if/else、switch）时，为两个路径编写测试
  - 永远不要提交使现有测试失败的代码

### B8. 提交

```bash
git status --porcelain
```

仅在有更改时提交。暂存所有引导文件（配置、测试目录、TESTING.md、CLAUDE.md、.github/workflows/test.yml（如果创建））：
`git commit -m "chore: bootstrap test framework ({framework name})"`

---

**创建输出目录：**

```bash
mkdir -p .gstack/qa-reports/screenshots
```

---

## 测试计划上下文

在回退到git差异启发式之前，检查更丰富的测试计划来源：

1. **项目范围的测试计划：** 检查 `~/.gstack/projects/` 中此仓库的最近 `*-test-plan-*.md` 文件
   ```bash
   eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
   ls -t ~/.gstack/projects/$SLUG/*-test-plan-*.md 2>/dev/null | head -1
   ```
2. **对话上下文：** 检查此对话中是否有先前的 `/plan-eng-review` 或 `/plan-ceo-review` 产生了测试计划输出
3. **使用更丰富的来源。** 仅当两者都不可用时才回退到git差异分析。

---

## 阶段1-6：QA基线

## 模式

### 差异感知（在功能分支上无URL时自动）

这是开发人员验证其工作的**主要模式**。当用户在没有URL的情况下说 `/qa` 且仓库在功能分支上时，自动：

1. **分析分支差异**以了解发生了什么变化：
   ```bash
   git diff main...HEAD --name-only
   git log main..HEAD --oneline
   ```

2. **从更改的文件中识别受影响的页面/路由**：
   - 控制器/路由文件 → 它们服务哪些URL路径
   - 视图/模板/组件文件 → 哪些页面渲染它们
   - 模型/服务文件 → 哪些页面使用这些模型（检查引用它们的控制器）
   - CSS/样式文件 → 哪些页面包含这些样式表
   - API端点 → 直接使用 `$B js "await fetch('/api/...')"` 测试它们
   - 静态页面（markdown、HTML）→ 直接导航到它们

3. **检测运行中的应用** — 检查常见的本地开发端口：
   ```bash
   $B goto http://localhost:3000 2>/dev/null && echo "在:3000找到应用" || \
   $B goto http://localhost:4000 2>/dev/null && echo "在:4000找到应用" || \
   $B goto http://localhost:8080 2>/dev/null && echo "在:8080找到应用"
   ```
   如果未找到本地应用，检查PR或环境中的暂存/预览URL。如果什么都不起作用，询问用户URL。

4. **测试每个受影响的页面/路由：**
   - 导航到页面
   - 截图
   - 检查控制台错误
   - 如果更改是交互式的（表单、按钮、流程），端到端测试交互
   - 使用 `snapshot -D` 在操作前后验证更改是否产生了预期效果

5. **与提交消息和PR描述交叉引用**以了解*意图* — 更改应该做什么？验证它实际上做到了。

6. **检查TODOS.md**（如果存在）以了解与更改文件相关的已知bug或问题。如果TODO描述了此分支应修复的bug，将其添加到测试计划中。如果在QA期间发现不在TODOS.md中的新bug，在报告中注明。

7. **报告发现**，范围限定在分支更改：
   - "测试的更改：此分支影响的N个页面/路由"
   - 对于每个：它工作吗？截图证据。
   - 相邻页面有任何回归吗？

**如果用户在差异感知模式下提供URL：** 使用该URL作为基础，但仍然将测试范围限定在更改的文件。

### 完整（提供URL时的默认值）
系统探索。访问每个可达页面。记录5-10个有充分证据的问题。生成健康评分。根据应用大小，需要5-15分钟。

### 快速（`--quick`）
30秒冒烟测试。访问主页 + 前5个导航目标。检查：页面加载？控制台错误？损坏的链接？生成健康评分。无详细问题文档。

### 回归（`--regression <baseline>`）
运行完整模式，然后加载先前运行的 `baseline.json`。差异：哪些问题已修复？哪些是新的？分数差异是多少？将回归部分追加到报告中。

---

## 工作流

### 阶段1：初始化

1. 找到browse二进制文件（见上面的设置）
2. 创建输出目录
3. 从 `qa/templates/qa-report-template.md` 复制报告模板到输出目录
4. 启动计时器以跟踪持续时间

### 阶段2：认证（如果需要）

**如果用户指定了认证凭据：**

```bash
$B goto <login-url>
$B snapshot -i                    # 找到登录表单
$B fill @e3 "user@example.com"
$B fill @e4 "[已编辑]"         # 报告中永远不要包含真实密码
$B click @e5                      # 提交
$B snapshot -D                    # 验证登录成功
```

**如果用户提供了cookie文件：**

```bash
$B cookie-import cookies.json
$B goto <target-url>
```

**如果需要2FA/OTP：** 询问用户代码并等待。

**如果CAPTCHA阻止你：** 告诉用户："请在浏览器中完成CAPTCHA，然后告诉我继续。"

### 阶段3：定向

获取应用程序的地图：

```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links                          # 映射导航结构
$B console --errors               # 登录页面有任何错误？
```

**检测框架**（在报告元数据中注明）：
- HTML中的 `__next` 或 `_next/data` 请求 → Next.js
- `csrf-token` meta标签 → Rails
- URL中的 `wp-content` → WordPress
- 无页面重载的客户端路由 → SPA

**对于SPA：** `links` 命令可能返回很少的结果，因为导航是客户端的。使用 `snapshot -i` 查找导航元素（按钮、菜单项）。

### 阶段4：探索

系统地访问页面。在每个页面：

```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page-name.png"
$B console --errors
```

然后按照**每页探索清单**（见 `qa/references/issue-taxonomy.md`）：

1. **视觉扫描** — 查看带注释的截图以查找布局问题
2. **交互元素** — 点击按钮、链接、控件。它们工作吗？
3. **表单** — 填写并提交。测试空、无效、边缘情况
4. **导航** — 检查所有进出路径
5. **状态** — 空状态、加载、错误、溢出
6. **控制台** — 交互后有任何新的JS错误？
7. **响应性** — 如果相关，检查移动视口：
   ```bash
   $B viewport 375x812
   $B screenshot "$REPORT_DIR/screenshots/page-mobile.png"
   $B viewport 1280x720
   ```

**深度判断：** 在核心功能（主页、仪表板、结账、搜索）上花更多时间，在次要页面（关于、条款、隐私）上花更少时间。

**快速模式：** 仅访问定向阶段的主页 + 前5个导航目标。跳过每页清单 — 只检查：加载？控制台错误？可见的损坏链接？

### 阶段5：记录

**发现每个问题后立即记录** — 不要批量处理。

**两个证据层级：**

**交互bug**（损坏的流程、无效按钮、表单失败）：
1. 在操作前截图
2. 执行操作
3. 截图显示结果
4. 使用 `snapshot -D` 显示发生了什么变化
5. 编写引用截图的复现步骤

```bash
$B screenshot "$REPORT_DIR/screenshots/issue-001-step-1.png"
$B click @e5
$B screenshot "$REPORT_DIR/screenshots/issue-001-result.png"
$B snapshot -D
```

**静态bug**（拼写错误、布局问题、缺失图像）：
1. 拍摄单个带注释的截图显示问题
2. 描述问题

```bash
$B snapshot -i -a -o "$REPORT_DIR/screenshots/issue-002.png"
```

**立即将每个问题写入报告**，使用 `qa/templates/qa-report-template.md` 中的模板格式。

### 阶段6：总结

1. **计算健康评分**使用下面的评分标准
2. **编写"前3个要修复的问题"** — 3个最高严重性的问题
3. **编写控制台健康摘要** — 汇总所有页面看到的控制台错误
4. **更新摘要表中的严重性计数**
5. **填写报告元数据** — 日期、持续时间、访问的页面、截图计数、框架
6. **保存基线** — 写入 `baseline.json`，包含：
   ```json
   {
     "date": "YYYY-MM-DD",
     "url": "<target>",
     "healthScore": N,
     "issues": [{ "id": "ISSUE-001", "title": "...", "severity": "...", "category": "..." }],
     "categoryScores": { "console": N, "links": N, ... }
   }
   ```

**回归模式：** 写入报告后，加载基线文件。比较：
- 健康评分差异
- 已修复的问题（在基线中但不在当前中）
- 新问题（在当前中但不在基线中）
- 将回归部分追加到报告

---

## 健康评分标准

计算每个类别得分（0-100），然后取加权平均值。

### 控制台（权重：15%）
- 0个错误 → 100
- 1-3个错误 → 70
- 4-10个错误 → 40
- 10+个错误 → 10

### 链接（权重：10%）
- 0个损坏 → 100
- 每个损坏链接 → -15（最低0）

### 按类别评分（视觉、功能、UX、内容、性能、可访问性）
每个类别从100开始。按发现扣分：
- 关键问题 → -25
- 高问题 → -15
- 中问题 → -8
- 低问题 → -3
每个类别最低0。

### 权重
| 类别 | 权重 |
|------|------|
| 控制台 | 15% |
| 链接 | 10% |
| 视觉 | 10% |
| 功能 | 20% |
| UX | 15% |
| 性能 | 10% |
| 内容 | 5% |
| 可访问性 | 15% |

### 最终得分
`score = Σ (category_score × weight)`

---

## 框架特定指南

### Next.js
- 检查控制台中的水合错误（`Hydration failed`，`Text content did not match`）
- 监控网络中的 `_next/data` 请求 — 404表示损坏的数据获取
- 测试客户端导航（点击链接，不仅仅是 `goto`）— 捕获路由问题
- 检查具有动态内容的页面的CLS（累积布局偏移）

### Rails
- 检查控制台中的N+1查询警告（如果是开发模式）
- 验证表单中CSRF令牌的存在
- 测试Turbo/Stimulus集成 — 页面过渡是否平滑？
- 检查flash消息是否正确显示和消失

### WordPress
- 检查插件冲突（来自不同插件的JS错误）
- 验证登录用户的管理栏可见性
- 测试REST API端点（`/wp-json/`）
- 检查混合内容警告（WP常见）

### 通用SPA（React、Vue、Angular）
- 使用 `snapshot -i` 进行导航 — `links` 命令会错过客户端路由
- 检查陈旧状态（导航离开并返回 — 数据是否刷新？）
- 测试浏览器后退/前进 — 应用是否正确处理历史？
- 检查内存泄漏（长时间使用后监控控制台）

---

## 重要规则

1. **复现是一切。** 每个问题至少需要一个截图。无例外。
2. **记录前验证。** 重试问题一次以确认它是可重现的，不是偶然。
3. **永远不要包含凭据。** 在复现步骤中为密码写入 `[已编辑]`。
4. **增量写入。** 找到问题后立即将其追加到报告。不要批量处理。
5. **永远不要阅读源代码。** 作为用户测试，不是开发人员。
6. **每次交互后检查控制台。** 视觉上不显示的JS错误仍然是bug。
7. **像用户一样测试。** 使用现实数据。端到端完成完整工作流。
8. **深度胜于广度。** 5-10个有充分文档和证据的问题 > 20个模糊描述。
9. **永远不要删除输出文件。** 截图和报告累积 — 这是有意的。
10. **对复杂UI使用 `snapshot -C`。** 找到可访问性树错过的可点击div。
11. **向用户显示截图。** 在每个 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 命令后，使用Read工具读取输出文件，以便用户可以内联看到它们。对于 `responsive`（3个文件），读取所有三个。这是关键 — 没有它，截图对用户是不可见的。

在阶段6结束时记录基线健康评分。

---

## 输出结构

```
.gstack/qa-reports/
├── qa-report-{domain}-{YYYY-MM-DD}.md    # 结构化报告
├── screenshots/
│   ├── initial.png                        # 登录页面带注释截图
│   ├── issue-001-step-1.png               # 每个问题的证据
│   ├── issue-001-result.png
│   ├── issue-001-before.png               # 修复前（如果已修复）
│   ├── issue-001-after.png                # 修复后（如果已修复）
│   └── ...
└── baseline.json                          # 用于回归模式
```

报告文件名使用域名和日期：`qa-report-myapp-com-2026-03-12.md`

---

## 阶段7：分类

按严重性排序所有发现的问题，然后根据所选层级决定修复哪些：

- **快速：** 仅修复关键 + 高。将中/低标记为"推迟"。
- **标准：** 修复关键 + 高 + 中。将低标记为"推迟"。
- **详尽：** 修复所有，包括外观/低严重性。

将无法从源代码修复的问题（例如，第三方小部件bug、基础设施问题）标记为"推迟"，无论层级如何。

---

## 阶段8：修复循环

对于每个可修复的问题，按严重性顺序：

### 8a. 定位源

```bash
# 搜索错误消息、组件名称、路由定义
# 全局搜索与受影响页面匹配的文件模式
```

- 找到负责bug的源文件
- **仅修改与问题直接相关的文件**

### 8b. 修复

- 阅读源代码，理解上下文
- 进行**最小修复** — 解决问题的最小更改
- **不要**重构周围代码、添加功能或"改进"无关的内容

### 8c. 提交

```bash
git add <only-changed-files>
git commit -m "fix(qa): ISSUE-NNN — 简短描述"
```

- 每个修复一次提交。永远不要捆绑多个修复。
- 消息格式：`fix(qa): ISSUE-NNN — 简短描述`

### 8d. 重新测试

- 导航回受影响的页面
- 拍摄**前后截图对**
- 检查控制台错误
- 使用 `snapshot -D` 验证更改产生了预期效果

```bash
$B goto <affected-url>
$B screenshot "$REPORT_DIR/screenshots/issue-NNN-after.png"
$B console --errors
$B snapshot -D
```

### 8e. 分类

- **已验证**：重新测试确认修复有效，未引入新错误
- **尽力而为**：应用了修复但无法完全验证（例如，需要认证状态、外部服务）
- **已回退**：检测到回归 → `git revert HEAD` → 将问题标记为"推迟"

### 8e.5. 回归测试

如果以下情况跳过：分类不是"已验证"，或者修复纯粹是视觉/CSS，没有JS行为，或者未检测到测试框架且用户拒绝引导。

**1. 研究项目的现有测试模式：**

阅读最接近修复的2-3个测试文件（相同目录，相同代码类型）。完全匹配：
- 文件命名、导入、断言风格、describe/it嵌套、设置/拆卸模式
回归测试必须看起来像是由同一开发者编写的。

**2. 跟踪bug的代码路径，然后编写回归测试：**

在编写测试之前，跟踪通过你刚刚修复的代码的数据流：
- 什么输入/状态触发了bug？（确切的前提条件）
- 它遵循什么代码路径？（哪些分支，哪些函数调用）
- 它在哪里中断？（失败的确切行/条件）
- 还有什么其他输入可能命中相同的代码路径？（修复周围的边缘情况）

测试**必须**：
- 设置触发bug的前提条件（使其中断的确切状态）
- 执行暴露bug的操作
- 断言正确行为（不是"它渲染"或"它不抛出"）
- 如果你在跟踪时发现相邻的边缘情况，也测试它们（例如，null输入、空数组、边界值）
- 包含完整的归因注释：
  ```
  // 回归：ISSUE-NNN — {什么损坏了}
  // 由 /qa 在 {YYYY-MM-DD} 发现
  // 报告：.gstack/qa-reports/qa-report-{domain}-{date}.md
  ```

测试类型决定：
- 控制台错误 / JS异常 / 逻辑bug → 单元或集成测试
- 损坏的表单 / API失败 / 数据流bug → 带有请求/响应的集成测试
- 带有JS行为的视觉bug（损坏的下拉菜单、动画）→ 组件测试
- 纯CSS → 跳过（由QA重新运行捕获）

生成单元测试。模拟所有外部依赖（DB、API、Redis、文件系统）。

使用自动递增的名称以避免冲突：检查现有的 `{name}.regression-*.test.{ext}` 文件，取最大编号 + 1。

**3. 仅运行新测试文件：**

```bash
{detected test command} {new-test-file}
```

**4. 评估：**
- 通过 → 提交：`git commit -m "test(qa): regression test for ISSUE-NNN — {desc}"`
- 失败 → 修复测试一次。仍然失败 → 删除测试，推迟。
- 探索时间 >2分钟 → 跳过并推迟。

**5. WTF可能性排除：** 测试提交不计入启发式。

### 8f. 自我调节（停止并评估）

每5个修复后（或任何回退后），计算WTF可能性：

```
WTF可能性：
  从0%开始
  每次回退：                +15%
  每个触及>3个文件的修复： +5%
  修复15后：               +1% 每个额外修复
  所有剩余低严重性：        +10%
  触及无关文件：           +20%
```

**如果WTF > 20%：** 立即停止。向用户展示你到目前为止所做的工作。询问是否继续。

**硬上限：50个修复。** 50个修复后，无论剩余多少问题，都停止。

---

## 阶段9：最终QA

应用所有修复后：

1. 重新运行所有受影响页面的QA
2. 计算最终健康评分
3. **如果最终评分比基线更差：** 突出警告 — 发生了回归

---

## 阶段10：报告

将报告写入本地和项目范围的位置：

**本地：** `.gstack/qa-reports/qa-report-{domain}-{YYYY-MM-DD}.md`

**项目范围：** 为跨会话上下文写入测试结果工件：
```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mkdir -p ~/.gstack/projects/$SLUG
```
写入 `~/.gstack/projects/{slug}/{user}-{branch}-test-outcome-{datetime}.md`

**每个问题的添加**（超出标准报告模板）：
- 修复状态：已验证 / 尽力而为 / 已回退 / 推迟
- 提交SHA（如果已修复）
- 更改的文件（如果已修复）
- 前后截图（如果已修复）

**摘要部分：**
- 发现的总问题数
- 应用的修复（已验证：X，尽力而为：Y，已回退：Z）
- 推迟的问题
- 健康评分差异：基线 → 最终

**PR摘要：** 包含适合PR描述的单行摘要：
> "QA发现N个问题，修复M个，健康评分X → Y。"

---

## 阶段11：TODOS.md更新

如果仓库有 `TODOS.md`：

1. **新推迟的bug** → 添加为带有严重性、类别和复现步骤的TODO
2. **已修复的bug在TODOS.md中** → 注释为"由/qa在{branch}，{date}修复"

---

## 附加规则（qa特定）

11. **需要干净的工作树。** 如果 `git status --porcelain` 非空，拒绝开始。
12. **每个修复一次提交。** 永远不要将多个修复捆绑到一个提交中。
13. **仅在第8e.5阶段生成回归测试时修改测试。** 永远不要修改CI配置。永远不要修改现有测试 — 只创建新测试文件。
14. **回归时回退。** 如果修复使情况更糟，立即 `git revert HEAD`。
15. **自我调节。** 遵循WTF可能性启发式。如有疑问，停止并询问。