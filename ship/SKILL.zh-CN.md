---
name: ship
version: 1.0.0
description: |
  发布工作流：检测 + 合并基础分支，运行测试，审查差异，更新版本号，更新CHANGELOG，提交，推送，创建PR。当被要求"发布"、"部署"、"推送到main"、"创建PR"或"合并并推送"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
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

# 发布：完全自动化发布工作流

你正在运行 `/ship` 工作流。这是一个 **非交互式、完全自动化** 的工作流。在任何步骤都不要要求确认。用户说 `/ship` 意味着执行它。直接运行并在最后输出PR URL。

**仅在以下情况停止：**
- 在基础分支上（中止）
- 无法自动解决的合并冲突（停止，显示冲突）
- 测试失败（停止，显示失败）
- 预落地审查发现需要用户判断的ASK项目
- 需要MINOR或MAJOR版本更新（询问 — 见步骤4）
- 需要用户决策的Greptile审查评论（复杂修复，假阳性）
- TODOS.md缺失且用户想要创建一个（询问 — 见步骤5.5）
- TODOS.md混乱且用户想要重新组织（询问 — 见步骤5.5）

**永远不要因以下原因停止：**
- 未提交的更改（总是包含它们）
- 版本更新选择（自动选择MICRO或PATCH — 见步骤4）
- CHANGELOG内容（从差异自动生成）
- 提交消息批准（自动提交）
- 多文件更改集（自动拆分为可二分查找的提交）
- TODOS.md已完成项目检测（自动标记）
- 可自动修复的审查发现（死代码，N+1，过时注释 — 自动修复）
- 测试覆盖缺口（自动生成并提交，或在PR正文中标记）

---

## 步骤1：飞行前检查

1. 检查当前分支。如果在基础分支或仓库的默认分支上，**中止**："你在基础分支上。从功能分支发布。"

2. 运行 `git status`（永远不要使用 `-uall`）。未提交的更改总是包含在内 — 无需询问。

3. 运行 `git diff <base>...HEAD --stat` 和 `git log <base>..HEAD --oneline` 以了解正在发布的内容。

4. 检查审查准备情况：

## 审查准备仪表板

完成审查后，阅读审查日志和配置以显示仪表板。

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
cat ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl 2>/dev/null || echo "NO_REVIEWS"
echo "---CONFIG---"
~/.claude/skills/gstack/bin/gstack-config get skip_eng_review 2>/dev/null || echo "false"
```

解析输出。找到每个技能（plan-ceo-review、plan-eng-review、plan-design-review、design-review-lite）的最近条目。忽略时间戳超过7天的条目。对于设计审查，显示 `plan-design-review`（完整视觉审计）和 `design-review-lite`（代码级检查）中较新的一个。在状态后附加"(FULL)"或"(LITE)"以区分。显示：

```
+====================================================================+
|                    审查准备仪表板                                 |
+====================================================================+
| 审查          | 运行次数 | 最后运行时间         | 状态     | 必需 |
|-----------------|------|---------------------|-----------|----------|
| 工程审查      |  1   | 2026-03-16 15:00    | 清洁     | 是      |
| CEO审查       |  0   | —                   | —         | 否       |
| 设计审查      |  0   | —                   | —         | 否       |
+--------------------------------------------------------------------+
|  verdict: 已清除 — 工程审查通过                                 |
+====================================================================+
```

**审查层级：**
- **工程审查（默认必需）：** 唯一控制发布的审查。涵盖架构、代码质量、测试、性能。可以通过 `gstack-config set skip_eng_review true` 全局禁用（"不要打扰我"设置）。
- **CEO审查（可选）：** 使用你的判断。对于大型产品/业务变更、新的用户面向功能或范围决策，推荐使用。对于错误修复、重构、基础设施和清理，跳过。
- **设计审查（可选）：** 使用你的判断。对于UI/UX变更，推荐使用。对于仅后端、基础设施或仅提示变更，跳过。

**判断逻辑：**
- **已清除**：工程审查在7天内有≥1个状态为"clean"的条目（或 `skip_eng_review` 为 `true`）
- **未清除**：工程审查缺失、过时（>7天）或有未解决的问题
- CEO和设计审查显示为上下文，但永远不会阻止发布
- 如果 `skip_eng_review` 配置为 `true`，工程审查显示"已跳过（全局）"，判断为已清除

如果工程审查不是"清洁"：

1. **检查此分支上的先前覆盖：**
   ```bash
   eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
   grep '"skill":"ship-review-override"' ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl 2>/dev/null || echo "NO_OVERRIDE"
   ```
   如果存在覆盖，显示仪表板并注明"审查门控先前已接受 — 继续。" 不要再询问。

2. **如果不存在覆盖，** 使用AskUserQuestion：
   - 显示工程审查缺失或有未解决的问题
   - 推荐：如果更改明显微不足道（< 20行，拼写修复，仅配置）选择C；对于较大更改选择B
   - 选项：A) 无论如何发布 B) 中止 — 先运行 /plan-eng-review C) 更改太小，不需要工程审查
   - 如果CEO审查缺失，作为信息提及（"CEO审查未运行 — 产品更改建议运行"）但不阻止
   - 对于设计审查：运行 `eval $(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)`。如果 `SCOPE_FRONTEND=true` 且仪表板中不存在设计审查（plan-design-review或design-review-lite），提及："设计审查未运行 — 此PR更改前端代码。轻量级设计检查将在步骤3.5自动运行，但考虑运行 /design-review 进行完整的视觉审计后实现。" 仍然永远不阻止。

3. **如果用户选择A或C，** 保存决策，以便此分支上未来的 `/ship` 运行跳过门控：
   ```bash
   eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
   echo '{"skill":"ship-review-override","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","decision":"USER_CHOICE"}' >> ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl
   ```
   将USER_CHOICE替换为"ship_anyway"或"not_relevant"。

---

## 步骤2：合并基础分支（测试前）

获取并将基础分支合并到功能分支，以便测试在合并状态下运行：

```bash
git fetch origin <base> && git merge origin/<base> --no-edit
```

**如果存在合并冲突：** 如果冲突简单（VERSION、schema.rb、CHANGELOG排序），尝试自动解决。如果冲突复杂或模糊，**停止**并显示它们。

**如果已经是最新的：** 静默继续。

---

## 步骤2.5：测试框架引导

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

---

## 步骤3：运行测试（在合并代码上）

**不要运行 `RAILS_ENV=test bin/rails db:migrate`** — `bin/test-lane` 已经在内部调用
`db:test:prepare`，它将架构加载到正确的lane数据库中。
在没有INSTANCE的情况下运行裸测试迁移会命中孤立的DB并损坏structure.sql。

并行运行两个测试套件：

```bash
bin/test-lane 2>&1 | tee /tmp/ship_tests.txt &
npm run test 2>&1 | tee /tmp/ship_vitest.txt &
wait
```

两个都完成后，读取输出文件并检查通过/失败。

**如果任何测试失败：** 显示失败并 **停止**。不要继续。

**如果全部通过：** 静默继续 — 只需简要记录计数。

---

## 步骤3.25：评估套件（条件）

当提示相关文件更改时，评估是强制性的。如果差异中没有提示文件，完全跳过此步骤。

**1. 检查差异是否触及提示相关文件：**

```bash
git diff origin/<base> --name-only
```

与这些模式匹配（来自CLAUDE.md）：
- `app/services/*_prompt_builder.rb`
- `app/services/*_generation_service.rb`，`*_writer_service.rb`，`*_designer_service.rb`
- `app/services/*_evaluator.rb`，`*_scorer.rb`，`*_classifier_service.rb`，`*_analyzer.rb`
- `app/services/concerns/*voice*.rb`，`*writing*.rb`，`*prompt*.rb`，`*token*.rb`
- `app/services/chat_tools/*.rb`，`app/services/x_thread_tools/*.rb`
- `config/system_prompts/*.txt`
- `test/evals/**/*`（评估基础设施更改影响所有套件）

**如果没有匹配：** 打印"无提示相关文件更改 — 跳过评估。" 并继续到步骤3.5。

**2. 识别受影响的评估套件：**

每个评估运行器（`test/evals/*_eval_runner.rb`）声明 `PROMPT_SOURCE_FILES`，列出哪些源文件影响它。搜索这些以找到哪些套件匹配更改的文件：

```bash
grep -l "changed_file_basename" test/evals/*_eval_runner.rb
```

映射运行器 → 测试文件：`post_generation_eval_runner.rb` → `post_generation_eval_test.rb`。

**特殊情况：**
- 对 `test/evals/judges/*.rb`、`test/evals/support/*.rb` 或 `test/evals/fixtures/` 的更改影响使用这些judges/support文件的所有套件。检查评估测试文件中的导入以确定哪些。
- 对 `config/system_prompts/*.txt` 的更改 — 搜索评估运行器以查找提示文件名，找到受影响的套件。
- 如果不确定哪些套件受影响，运行所有可能受影响的套件。过度测试比错过回归更好。

**3. 以 `EVAL_JUDGE_TIER=full` 运行受影响的套件：**

`/ship` 是预合并门控，因此始终使用完整层级（Sonnet结构 + Opus角色judges）。

```bash
EVAL_JUDGE_TIER=full EVAL_VERBOSE=1 bin/test-lane --eval test/evals/<suite>_eval_test.rb 2>&1 | tee /tmp/ship_evals.txt
```

如果需要运行多个套件，按顺序运行它们（每个都需要测试lane）。如果第一个套件失败，立即停止 — 不要在剩余套件上浪费API成本。

**4. 检查结果：**

- **如果任何评估失败：** 显示失败、成本仪表板，并 **停止**。不要继续。
- **如果全部通过：** 记录通过计数和成本。继续到步骤3.5。

**5. 保存评估输出** — 在PR正文（步骤8）中包含评估结果和成本仪表板。

**层级参考（供上下文 — /ship 始终使用 `full`）：**
| 层级 | 何时 | 速度（缓存） | 成本 |
|------|------|----------------|------|
| `fast`（Haiku） | 开发迭代，冒烟测试 | ~5s（快14倍） | ~$0.07/运行 |
| `standard`（Sonnet） | 默认开发，`bin/test-lane --eval` | ~17s（快4倍） | ~$0.37/运行 |
| `full`（Opus角色） | **`/ship` 和预合并** | ~72s（基线） | ~$1.27/运行 |

---

## 步骤3.4：测试覆盖率审计

100%覆盖率是目标 — 每个未测试的路径都是bug隐藏的地方，氛围编码变成yolo编码。评估实际编码的内容（来自差异），而不是计划的内容。

**0. 测试计数前后：**

```bash
# 生成前计数测试文件
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' | grep -v node_modules | wc -l
```

将此数字存储在PR正文中。

**1. 使用 `git diff origin/<base>...HEAD` 追踪每个更改的代码路径：**

阅读每个更改的文件。对于每个文件，追踪数据如何流经代码 — 不要只是列出函数，实际跟随执行：

1. **阅读差异。** 对于每个更改的文件，阅读完整文件（不仅仅是差异块）以理解上下文。
2. **追踪数据流。** 从每个入口点（路由处理程序、导出函数、事件监听器、组件渲染）开始，跟随数据通过每个分支：
   - 输入来自哪里？（请求参数、props、数据库、API调用）
   - 什么转换它？（验证、映射、计算）
   - 它去哪里？（数据库写入、API响应、渲染输出、副作用）
   - 每个步骤可能出什么问题？（null/undefined、无效输入、网络故障、空集合）
3. **绘制执行图。** 对于每个更改的文件，绘制ASCII图显示：
   - 每个添加或修改的函数/方法
   - 每个条件分支（if/else、switch、三元、保护子句、早期返回）
   - 每个错误路径（try/catch、rescue、错误边界、回退）
   - 每个对另一个函数的调用（追踪到它 — 它是否有未测试的分支？）
   - 每个边缘：null输入、空数组、无效类型会发生什么？

这是关键步骤 — 你正在构建一张地图，显示基于输入可能执行不同的每一行代码。此图中的每个分支都需要测试。

**2. 映射用户流程、交互和错误状态：**

代码覆盖率是不够的 — 你需要覆盖真实用户如何与更改的代码交互。对于每个更改的功能，思考：

- **用户流程：** 用户采取什么动作序列会触及此代码？映射完整旅程（例如，"用户点击'支付' → 表单验证 → API调用 → 成功/失败屏幕"）。旅程的每个步骤都需要测试。
- **交互边缘情况：** 当用户做意外的事情时会发生什么？
  - 双击/快速重新提交
  - 操作中途导航离开（后退按钮、关闭标签、点击另一个链接）
  - 使用陈旧数据提交（页面打开30分钟，会话过期）
  - 慢速连接（API花费10秒 — 用户看到什么？）
  - 并发操作（两个标签，同一个表单）
- **用户可见的错误状态：** 对于代码处理的每个错误，用户实际体验是什么？
  - 有明确的错误消息还是静默失败？
  - 用户可以恢复（重试、返回、修复输入）还是卡住？
  - 无网络、API 500、服务器返回无效数据会发生什么？
- **空/零/边界状态：** UI显示零结果、10,000个结果、单个字符输入、最大长度输入时会显示什么？

将这些添加到代码分支旁边的图表中。没有测试的用户流程与未测试的if/else一样是缺口。

**3. 检查每个分支是否有现有测试：**

逐个分支检查你的图表 — 代码路径和用户流程。对于每个分支，搜索测试：
- 函数 `processPayment()` → 查找 `billing.test.ts`、`billing.spec.ts`、`test/billing_test.rb`
- if/else → 查找覆盖true和false路径的测试
- 错误处理程序 → 查找触发该特定错误条件的测试
- 对 `helperFn()` 的调用，它有自己的分支 → 那些分支也需要测试
- 用户流程 → 查找走过旅程的集成或E2E测试
- 交互边缘情况 → 查找模拟意外动作的测试

质量评分标准：
- ★★★ 测试行为，包括边缘情况和错误路径
- ★★   测试正确行为，仅愉快路径
- ★    冒烟测试 / 存在检查 / 微不足道的断言（例如，"它渲染"，"它不抛出"）

**4. 输出ASCII覆盖图：**

在同一图中包含代码路径和用户流程：

```
代码路径覆盖
===========================
[+] src/services/billing.ts
    │
    ├── processPayment()
    │   ├── [★★★ 已测试] 愉快路径 + 卡被拒绝 + 超时 — billing.test.ts:42
    │   ├── [缺口]         网络超时 — 无测试
    │   └── [缺口]         无效货币 — 无测试
    │
    └── refundPayment()
        ├── [★★  已测试] 全额退款 — billing.test.ts:89
        └── [★   已测试] 部分退款（仅检查非抛出） — billing.test.ts:101

用户流程覆盖
===========================
[+] 支付结账流程
    │
    ├── [★★★ 已测试] 完成购买 — checkout.e2e.ts:15
    ├── [缺口]         双击提交 — 无测试
    ├── [缺口]         支付过程中导航离开 — 无测试
    └── [★   已测试] 表单验证错误（仅检查渲染） — checkout.test.ts:40

[+] 错误状态
    │
    ├── [★★  已测试] 卡被拒绝消息 — billing.test.ts:58
    ├── [缺口]         网络超时UX（用户看到什么？） — 无测试
    └── [缺口]         空购物车提交 — 无测试

─────────────────────────────────
覆盖：12个路径中的5个已测试（42%）
  代码路径：5个中的3个（60%）
  用户流程：7个中的2个（29%）
质量：  ★★★: 2  ★★: 2  ★: 1
缺口：7个路径需要测试
─────────────────────────────────
```

**快速路径：** 所有路径都被覆盖 → "步骤3.4：所有新代码路径都有测试覆盖 ✓" 继续。

**5. 为未覆盖的路径生成测试：**

如果检测到测试框架（或在步骤2.5中引导）：
- 首先优先考虑错误处理程序和边缘情况（愉快路径更可能已经测试）
- 阅读2-3个现有测试文件以完全匹配约定
- 生成单元测试。模拟所有外部依赖（DB、API、Redis）。
- 编写测试，使用真实断言测试特定未覆盖的路径
- 运行每个测试。通过 → 提交为 `test: coverage for {feature}`
- 失败 → 修复一次。仍然失败 → 恢复，在图表中注明缺口。

上限：最多30个代码路径，最多生成20个测试（代码 + 用户流程组合），每个测试探索上限2分钟。

如果没有测试框架且用户拒绝引导 → 仅图表，不生成。注意："测试生成跳过 — 未配置测试框架。"

**差异仅为测试更改：** 完全跳过步骤3.4："无新应用代码路径需要审计。"

**6. 后计数和覆盖摘要：**

```bash
# 生成后计数测试文件
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' | grep -v node_modules | wc -l
```

对于PR正文：`测试：{before} → {after} (+{delta} 新)`
覆盖行：`测试覆盖审计：N个新代码路径。M个已覆盖（X%）。生成K个测试，J个已提交。`

---

## 步骤3.5：预落地审查

审查差异以查找测试无法捕获的结构问题。

1. 阅读 `.claude/skills/review/checklist.md`。如果无法读取文件，**停止**并报告错误。

2. 运行 `git diff origin/<base>` 获取完整差异（范围限定为针对新获取的基础分支的功能更改）。

3. 分两次应用审查清单：
   - **第1次（关键）：** SQL & 数据安全，LLM输出信任边界
   - **第2次（信息性）：** 所有剩余类别

## 设计审查（条件，差异范围）

使用 `gstack-diff-scope` 检查差异是否触及前端文件：

```bash
eval $(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)
```

**如果 `SCOPE_FRONTEND=false`：** 静默跳过设计审查。无输出。

**如果 `SCOPE_FRONTEND=true`：**

1. **检查DESIGN.md。** 如果仓库根目录中存在 `DESIGN.md` 或 `design-system.md`，阅读它。所有设计发现都根据它进行校准 — DESIGN.md中认可的模式不会被标记。如果未找到，使用通用设计原则。

2. **阅读 `.claude/skills/review/design-checklist.md`。** 如果无法读取文件，跳过设计审查并附带说明："设计清单未找到 — 跳过设计审查。"

3. **阅读每个更改的前端文件**（完整文件，不仅仅是差异块）。前端文件由清单中列出的模式识别。

4. **应用设计清单** 到更改的文件。对于每个项目：
   - **[高] 机械CSS修复**（`outline: none`、`!important`、`font-size < 16px`）：分类为AUTO-FIX
   - **[高/中] 需要设计判断**：分类为ASK
   - **[低] 基于意图的检测**：呈现为"可能 — 请视觉验证或运行 /design-review"

5. **包括发现** 在审查输出的"设计审查"标题下，遵循清单中的输出格式。设计发现与代码审查发现合并到相同的Fix-First流程中。

6. **记录结果** 用于审查准备仪表板：

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mkdir -p ~/.gstack/projects/$SLUG
echo '{"skill":"design-review-lite","timestamp":"TIMESTAMP","status":"STATUS","findings":N,"auto_fixed":M}' >> ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl
```

替换：TIMESTAMP = ISO 8601日期时间，STATUS = 如果0个发现则为"clean"，否则为"issues_found"，N = 总发现数，M = 自动修复计数。

   将任何设计发现与代码审查发现一起包含。它们遵循下面相同的Fix-First流程。

4. **根据checklist.md中的Fix-First启发式** 将每个发现分类为AUTO-FIX或ASK。关键发现倾向于ASK；信息性发现倾向于AUTO-FIX。

5. **自动修复所有AUTO-FIX项目。** 应用每个修复。每个修复输出一行：
   `[已自动修复] [文件:行] 问题 → 你做了什么`

6. **如果还有ASK项目，** 在一个AskUserQuestion中呈现它们：
   - 列出每个项目的编号、严重性、问题、推荐修复
   - 每个项目的选项：A) 修复 B) 跳过
   - 总体推荐
   - 如果ASK项目为3个或更少，你可以使用单独的AskUserQuestion调用

7. **所有修复后（自动 + 用户批准）：**
   - 如果应用了任何修复：按名称提交修复的文件（`git add <fixed-files> && git commit -m "fix: pre-landing review fixes"`），然后 **停止** 并告诉用户再次运行 `/ship` 以重新测试。
   - 如果未应用修复（所有ASK项目都被跳过，或未发现问题）：继续到步骤4。

8. 输出摘要：`预落地审查：N个问题 — M个自动修复，K个询问（J个修复，L个跳过）`

   如果未发现问题：`预落地审查：未发现问题。`

保存审查输出 — 它会在步骤8中进入PR正文。

---

## 步骤3.75：处理Greptile审查评论（如果PR存在）

阅读 `.claude/skills/review/greptile-triage.md` 并遵循获取、过滤、分类和**升级检测**步骤。

**如果不存在PR、`gh`失败、API返回错误或没有Greptile评论：** 静默跳过此步骤。继续到步骤4。

**如果找到Greptile评论：**

在输出中包含Greptile摘要：`+ N个Greptile评论（X个有效，Y个已修复，Z个假阳性）`

在回复任何评论之前，运行greptile-triage.md中的**升级检测**算法，以确定使用Tier 1（友好）还是Tier 2（坚定）回复模板。

对于每个分类的评论：

**有效且可操作：** 使用AskUserQuestion：
- 评论（file:line或[顶级] + 正文摘要 + 永久链接URL）
- `推荐：选择A因为[一行理由]`
- 选项：A) 现在修复，B) 确认并继续发布，C) 这是假阳性
- 如果用户选择A：应用修复，提交修复的文件（`git add <fixed-files> && git commit -m "fix: address Greptile review — <brief description>"`），使用greptile-triage.md中的**修复回复模板**回复（包括内联差异 + 解释），并保存到项目和全局greptile-history（类型：fix）。
- 如果用户选择C：使用greptile-triage.md中的**假阳性回复模板**回复（包括证据 + 建议重新排名），保存到项目和全局greptile-history（类型：fp）。

**有效但已修复：** 使用greptile-triage.md中的**已修复回复模板**回复 — 不需要AskUserQuestion：
- 包括所做的工作和修复提交SHA
- 保存到项目和全局greptile-history（类型：already-fixed）

**假阳性：** 使用AskUserQuestion：
- 显示评论和你认为它错误的原因（file:line或[顶级] + 正文摘要 + 永久链接URL）
- 选项：
  - A) 回复Greptile解释假阳性（如果明显错误，推荐）
  - B) 无论如何修复（如果微不足道）
  - C) 静默忽略
- 如果用户选择A：使用greptile-triage.md中的**假阳性回复模板**回复（包括证据 + 建议重新排名），保存到项目和全局greptile-history（类型：fp）

**已抑制：** 静默跳过 — 这些是来自以前分类的已知假阳性。

**所有评论解决后：** 如果应用了任何修复，步骤3的测试现在已过时。**重新运行测试**（步骤3）然后再继续到步骤4。如果未应用修复，继续到步骤4。

---

## 步骤4：版本更新（自动决定）

1. 阅读当前 `VERSION` 文件（4位格式：`MAJOR.MINOR.PATCH.MICRO`）

2. **根据差异自动决定更新级别：**
   - 计算更改的行数（`git diff origin/<base>...HEAD --stat | tail -1`）
   - **MICRO**（第4位）：< 50行更改，微不足道的调整，拼写错误，配置
   - **PATCH**（第3位）：50+行更改，错误修复，中小型功能
   - **MINOR**（第2位）：**询问用户** — 仅用于主要功能或重大架构更改
   - **MAJOR**（第1位）：**询问用户** — 仅用于里程碑或破坏性更改

3. 计算新版本：
   - 增加一位数字会将其右侧的所有数字重置为0
   - 示例：`0.19.1.0` + PATCH → `0.19.2.0`

4. 将新版本写入 `VERSION` 文件。

---

## 步骤5：CHANGELOG（自动生成）

1. 阅读 `CHANGELOG.md` 头部以了解格式。

2. 从**分支上的所有提交**（不仅仅是最近的）自动生成条目：
   - 使用 `git log <base>..HEAD --oneline` 查看要发布的每个提交
   - 使用 `git diff <base>...HEAD` 查看与基础分支的完整差异
   - CHANGELOG条目必须全面覆盖PR中的所有更改
   - 如果分支上的现有CHANGELOG条目已经覆盖了一些提交，用新版本的一个统一条目替换它们
   - 将更改分类到适用的部分：
     - `### Added` — 新功能
     - `### Changed` — 对现有功能的更改
     - `### Fixed` — 错误修复
     - `### Removed` — 移除的功能
   - 编写简洁、描述性的项目符号
   - 插入文件头部（第5行）之后，日期为今天
   - 格式：`## [X.Y.Z.W] - YYYY-MM-DD`

**不要要求用户描述更改。** 从差异和提交历史推断。

---

## 步骤5.5：TODOS.md（自动更新）

将项目的TODOS.md与要发布的更改交叉引用。自动标记已完成的项目；仅在文件缺失或混乱时提示。

阅读 `.claude/skills/review/TODOS-format.md` 以获取规范格式参考。

**1. 检查TODOS.md是否存在** 在仓库根目录中。

**如果TODOS.md不存在：** 使用AskUserQuestion：
- 消息："GStack建议维护一个按技能/组件组织的TODOS.md，然后按优先级排序（顶部P0到P4，底部Completed）。有关完整格式，请参见TODOS-format.md。您是否要创建一个？"
- 选项：A) 现在创建，B) 现在跳过
- 如果A：创建 `TODOS.md` 骨架（# TODOS标题 + ## Completed部分）。继续到步骤3。
- 如果B：跳过步骤5.5的其余部分。继续到步骤6。

**2. 检查结构和组织：**

阅读TODOS.md并验证它遵循推荐的结构：
- 项目分组在 `## <Skill/Component>` 标题下
- 每个项目有 `**Priority:**` 字段，值为P0-P4
- 底部有 `## Completed` 部分

**如果混乱**（缺少优先级字段，无组件分组，无Completed部分）：使用AskUserQuestion：
- 消息："TODOS.md不遵循推荐的结构（技能/组件分组，P0-P4优先级，Completed部分）。您是否要重新组织它？"
- 选项：A) 现在重新组织（推荐），B) 保持原样
- 如果A：按照TODOS-format.md就地重新组织。保留所有内容 — 仅重组，永远不删除项目。
- 如果B：不重组，继续到步骤3。

**3. 检测已完成的TODO：**

此步骤完全自动 — 无用户交互。

使用之前步骤中收集的差异和提交历史：
- `git diff <base>...HEAD`（与基础分支的完整差异）
- `git log <base>..HEAD --oneline`（要发布的所有提交）

对于每个TODO项目，通过以下方式检查此PR中的更改是否完成它：
- 将提交消息与TODO标题和描述匹配
- 检查TODO中引用的文件是否出现在差异中
- 检查TODO描述的工作是否与功能更改匹配

**保守：** 只有当差异中有明确证据时，才将TODO标记为已完成。如果不确定，保持原样。

**4. 将已完成的项目** 移动到底部的 `## Completed` 部分。追加：`**Completed:** vX.Y.Z (YYYY-MM-DD)`

**5. 输出摘要：**
- `TODOS.md: N个项目标记为完成（item1, item2, ...）。M个项目剩余。`
- 或：`TODOS.md: 未检测到已完成的项目。M个项目剩余。`
- 或：`TODOS.md: 已创建。` / `TODOS.md: 已重组。`

**6. 防御性：** 如果无法写入TODOS.md（权限错误，磁盘已满），警告用户并继续。永远不要因TODOS失败而停止发布工作流。

保存此摘要 — 它会在步骤8中进入PR正文。

---

## 步骤6：提交（可二分查找的块）

**目标：** 创建小的、逻辑的提交，与 `git bisect` 配合良好，并帮助LLM理解更改。

1. 分析差异并将更改分组为逻辑提交。每个提交应代表**一个连贯的更改** — 不是一个文件，而是一个逻辑单元。

2. **提交顺序**（较早的提交在前）：
   - **基础设施：** 迁移、配置更改、路由添加
   - **模型和服务：** 新模型、服务、关注点（及其测试）
   - **控制器和视图：** 控制器、视图、JS/React组件（及其测试）
   - **VERSION + CHANGELOG + TODOS.md：** 总是在最终提交中

3. **拆分规则：**
   - 模型及其测试文件放在同一提交中
   - 服务及其测试文件放在同一提交中
   - 控制器、其视图和其测试放在同一提交中
   - 迁移是自己的提交（或与它们支持的模型分组）
   - 配置/路由更改可以与它们启用的功能分组
   - 如果总差异很小（< 50行，跨 < 4个文件），单个提交就可以

4. **每个提交必须独立有效** — 无损坏的导入，无对不存在的代码的引用。按依赖关系在前的顺序提交。

5. 撰写每个提交消息：
   - 第一行：`<type>: <summary>`（type = feat/fix/chore/refactor/docs）
   - 正文：简要描述此提交包含的内容
   - 只有**最终提交**（VERSION + CHANGELOG）获得版本标签和共同作者尾部：

```bash
git commit -m "$(cat <<'EOF'
chore: bump version and changelog (vX.Y.Z.W)

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## 步骤6.5：验证门控

**铁律：没有新鲜的验证证据，不要声称完成。**

推送前，如果步骤4-6期间代码更改，重新验证：

1. **测试验证：** 如果步骤3测试运行后有任何代码更改（审查发现的修复，CHANGELOG编辑不算），重新运行测试套件。粘贴新鲜输出。步骤3的陈旧输出**不可接受**。

2. **构建验证：** 如果项目有构建步骤，运行它。粘贴输出。

3. **防止合理化：**
   - "现在应该工作" → 运行它。
   - "我有信心" → 信心不是证据。
   - "我之前已经测试过" → 代码从那时起已更改。再次测试。
   - "这是微不足道的更改" → 微不足道的更改会破坏生产。

**如果测试在此处失败：** 停止。不要推送。修复问题并返回步骤3。

没有验证就声称工作完成是不诚实，不是效率。

---

## 步骤7：推送

推送到带有上游跟踪的远程：

```bash
git push -u origin <branch-name>
```

---

## 步骤8：创建PR

使用 `gh` 创建拉取请求：

```bash
gh pr create --base <base> --title "<type>: <summary>" --body "$(cat <<'EOF'
## 摘要
<来自CHANGELOG的项目符号>

## 测试覆盖
<来自步骤3.4的覆盖图，或"所有新代码路径都有测试覆盖。">
<如果运行了步骤3.4："测试：{before} → {after} (+{delta} 新)">

## 预落地审查
<来自步骤3.5代码审查的发现，或"未发现问题。">

## 设计审查
<如果运行了设计审查："设计审查（轻量级）：N个发现 — M个自动修复，K个跳过。AI Slop：clean/N个问题。">
<如果没有前端文件更改："没有前端文件更改 — 设计审查跳过。">

## 评估结果
<如果运行了评估：套件名称，通过/失败计数，成本仪表板摘要。如果跳过："无提示相关文件更改 — 评估跳过。">

## Greptile审查
<如果找到Greptile评论：带[已修复] / [假阳性] / [已修复]标签的项目符号列表 + 每个评论的一行摘要>
<如果未找到Greptile评论："无Greptile评论。">
<如果步骤3.75期间不存在PR：完全省略此部分>

## TODOS
<如果标记了已完成的项目：带版本的已完成项目的项目符号列表>
<如果没有项目完成："此PR中没有完成的TODO项目。">
<如果创建或重组了TODOS.md：注明>
<如果TODOS.md不存在且用户跳过：省略此部分>

## 测试计划
- [x] 所有Rails测试通过（N次运行，0次失败）
- [x] 所有Vitest测试通过（N个测试）

🤖 使用 [Claude Code](https://claude.com/claude-code) 生成
EOF
)"
```

**输出PR URL** — 这应该是用户看到的最终输出。

---

## 重要规则

- **永远不要跳过测试。** 如果测试失败，停止。
- **永远不要跳过预落地审查。** 如果checklist.md不可读，停止。
- **永远不要强制推送。** 仅使用常规 `git push`。
- **永远不要要求确认** 除了MINOR/MAJOR版本更新和预落地审查ASK项目（最多批量到一个AskUserQuestion）。
- **始终使用VERSION文件中的4位版本格式**。
- **CHANGELOG中的日期格式：** `YYYY-MM-DD`
- **拆分提交以实现可二分查找** — 每个提交 = 一个逻辑更改。
- **TODOS.md完成检测必须保守。** 只有当差异清楚显示工作完成时，才将项目标记为已完成。
- **使用greptile-triage.md中的Greptile回复模板。** 每个回复都包含证据（内联差异，代码引用，重新排名建议）。永远不要发布模糊的回复。
- **没有新鲜的验证证据，永远不要推送。** 如果步骤3测试后代码更改，推送前重新运行。
- **步骤3.4生成覆盖测试。** 它们必须在提交前通过。永远不要提交失败的测试。
- **目标是：用户说 `/ship`，他们看到的下一件事是审查 + PR URL。**