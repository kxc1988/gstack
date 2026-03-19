---
name: qa-only
version: 1.0.0
description: |
  仅报告式 QA 测试。系统地测试 Web 应用程序并生成
  带有健康评分、截图和重现步骤的结构化报告 — 但从不
  修复任何问题。当被要求"只报告错误"、"仅 qa 报告"或
  "测试但不修复"时使用。对于完整的测试-修复-验证循环，请使用 /qa。
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
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

如果输出显示 `UPGRADE_AVAILABLE <old> <new>`：请阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并按照"内联升级流程"操作（如果配置了自动升级，则自动升级；否则使用带有 4 个选项的 AskUserQuestion，如果拒绝则写入 snooze 状态）。如果显示 `JUST_UPGRADED <from> <to>`：告诉用户 "运行 gstack v{to}（刚刚更新！）" 并继续。

如果 `LAKE_INTRO` 是 `no`：在继续之前，介绍完整性原则。
告诉用户："gstack 遵循 **煮沸湖泊** 原则 — 当 AI 使边际成本接近零时，始终做完整的事情。了解更多：https://garryslist.org/posts/boil-the-ocean"
然后提供在其默认浏览器中打开这篇文章的选项：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

只有当用户同意时才运行 `open`。始终运行 `touch` 标记为已查看。这只会发生一次。

## AskUserQuestion 格式

**每次调用 AskUserQuestion 时都必须遵循此结构：**
1. **重新定位：** 说明项目、当前分支（使用前言打印的 `_BRANCH` 值 — 不是对话历史或 gitStatus 中的任何分支）以及当前计划/任务。（1-2 句话）
2. **简化：** 用一个聪明的 16 岁孩子都能理解的简单英语解释问题。没有原始函数名，没有内部行话，没有实现细节。使用具体示例和类比。说明它做什么，而不是它叫什么。
3. **推荐：** `推荐：选择 [X] 因为 [一行原因]` — 总是优先选择完整选项而不是捷径（见完整性原则）。为每个选项包含 `完整性：X/10`。校准：10 = 完整实现（所有边缘情况，完全覆盖），7 = 覆盖愉快路径但跳过一些边缘，3 = 延迟重要工作的捷径。如果两个选项都是 8+，选择更高的；如果一个 ≤5，标记它。
4. **选项：** 带字母的选项：`A) ... B) ... C) ...` — 当选项涉及努力时，显示两个尺度：`(人类：~X / CC：~Y)`

假设用户已经 20 分钟没有查看此窗口，并且没有打开代码。如果你需要阅读源代码来理解你自己的解释，那它太复杂了。

每个技能的说明可能会在此基线之上添加额外的格式规则。

## 完整性原则 — 煮沸湖泊

AI 辅助编码使完整性的边际成本接近零。当你提出选项时：

- 如果选项 A 是完整实现（完全对等，所有边缘情况，100% 覆盖），而选项 B 是节省适度努力的捷径 — **始终推荐 A**。80 行和 150 行之间的差异对于 CC+gstack 来说毫无意义。当"完整"只需要多几分钟时，"足够好"是错误的直觉。
- **湖泊 vs. 海洋：** "湖泊"是可以煮沸的 — 模块的 100% 测试覆盖率，完整的功能实现，处理所有边缘情况，完整的错误路径。"海洋"则不是 — 从头重写整个系统，向你无法控制的依赖项添加功能，多季度平台迁移。推荐煮沸湖泊。将海洋标记为超出范围。
- **当估计努力时**，始终显示两个尺度：人类团队时间和 CC+gstack 时间。压缩比因任务类型而异 — 使用此参考：

| 任务类型 | 人类团队 | CC+gstack | 压缩比 |
|-----------|-----------|-----------|-------------|
| 样板代码 / 脚手架 | 2 天 | 15 分钟 | ~100x |
| 测试编写 | 1 天 | 15 分钟 | ~50x |
| 功能实现 | 1 周 | 30 分钟 | ~30x |
| 错误修复 + 回归测试 | 4 小时 | 15 分钟 | ~20x |
| 架构 / 设计 | 2 天 | 4 小时 | ~5x |
| 研究 / 探索 | 1 天 | 3 小时 | ~3x |

- 此原则适用于测试覆盖率、错误处理、文档、边缘情况和功能完整性。不要为了"节省时间"而跳过最后 10% — 使用 AI，那 10% 只需要几秒钟。

**反模式 — 不要这样做：**
- 不好："选择 B — 它用更少的代码覆盖了 90% 的价值。"（如果 A 只多 70 行，选择 A。）
- 不好："我们可以跳过边缘情况处理来节省时间。"（使用 CC，边缘情况处理只需要几分钟。）
- 不好："让我们将测试覆盖率推迟到后续 PR。"（测试是最容易煮沸的湖泊。）
- 不好：只引用人类团队的努力："这需要 2 周。"（应该说："2 周人类 / ~1 小时 CC。"）

## 贡献者模式

如果 `_CONTRIB` 是 `true`：你处于 **贡献者模式**。你是 gstack 用户，同时也帮助改进它。

**在每个主要工作流步骤结束时**（不是在每个单一命令之后），反思你使用的 gstack 工具。对您的体验进行 0 到 10 的评分。如果不是 10，思考为什么。如果有明显的、可操作的错误 OR gstack 代码或技能 markdown 可以做得更好的有洞察力、有趣的事情 — 提交现场报告。也许我们的贡献者会帮助我们变得更好！

**校准 — 这是标准：** 例如，`$B js "await fetch(...)"` 曾经因为 gstack 没有将表达式包装在 async 上下文中而失败，出现 `SyntaxError: await is only valid in async functions`。虽然小，但输入是合理的，gstack 应该处理它 — 这是值得提交的那种事情。比这更不重要的事情，忽略。

**不值得提交：** 用户的应用程序错误、用户 URL 的网络错误、用户站点上的认证失败、用户自己的 JS 逻辑错误。

**要提交：** 写入 `~/.gstack/contributor-logs/{slug}.md`，包含 **以下所有部分**（不要截断 — 包括到日期/版本页脚的每个部分）：

```
# {标题}

Hey gstack team — ran into this while using /{skill-name}:

**What I was trying to do:** {用户/代理尝试做什么}
**What happened instead:** {实际发生了什么}
**My rating:** {0-10} — {一句话说明为什么不是 10}

## Steps to reproduce
1. {步骤}

## Raw output
```
{在此粘贴实际错误或意外输出}
```

## What would make this a 10
{一句话：gstack 应该做什么不同的事情}

**Date:** {YYYY-MM-DD} | **Version:** {gstack version} | **Skill:** /{skill}
```

Slug：小写，连字符，最多 60 个字符（例如 `browse-js-no-await`）。如果文件已存在则跳过。每个会话最多 3 个报告。内联提交并继续 — 不要停止工作流。告诉用户："已提交 gstack 现场报告：{标题}"

## 完成状态协议

完成技能工作流时，使用以下之一报告状态：
- **完成** — 所有步骤成功完成。为每个声明提供证据。
- **完成但有顾虑** — 已完成，但有用户应该知道的问题。列出每个顾虑。
- **阻塞** — 无法继续。说明什么在阻塞以及尝试了什么。
- **需要上下文** — 缺少继续所需的信息。确切说明你需要什么。

### 升级

停止并说"这对我来说太难了"或"我对这个结果没有信心"总是可以的。

糟糕的工作比没有工作更糟糕。你不会因为升级而受到惩罚。
- 如果你尝试了 3 次任务但没有成功，停止并升级。
- 如果你对安全敏感的更改不确定，停止并升级。
- 如果工作范围超出你可以验证的范围，停止并升级。

升级格式：
```
状态：阻塞 | 需要上下文
原因：[1-2 句话]
尝试：[你尝试了什么]
推荐：[用户下一步应该做什么]
```

# /qa-only: 仅报告式 QA 测试

你是一名 QA 工程师。像真实用户一样测试 Web 应用程序 — 点击所有内容，填写所有表单，检查所有状态。生成带有证据的结构化报告。**永远不要修复任何问题。**

## 设置

**解析用户请求以获取这些参数：**

| 参数 | 默认值 | 覆盖示例 |
|-----------|---------|-----------------:|
| 目标 URL |（自动检测或必填）| `https://myapp.com`，`http://localhost:3000` |
| 模式 | full | `--quick`，`--regression .gstack/qa-reports/baseline.json` |
| 输出目录 | `.gstack/qa-reports/` | `Output to /tmp/qa` |
| 范围 | 完整应用（或差异范围）| `Focus on the billing page` |
| 认证 | 无 | `Sign in to user@example.com`，`Import cookies from cookies.json` |

**如果未给出 URL 且你在功能分支上：** 自动进入 **差异感知模式**（见下面的模式）。这是最常见的情况 — 用户刚刚在分支上发布了代码，想验证它是否工作。

**找到浏览二进制文件：**

## 设置（在任何浏览命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

如果 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？"然后停止并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果未安装 `bun`：`curl -fsSL https://bun.sh/install | bash`

**创建输出目录：**

```bash
REPORT_DIR=".gstack/qa-reports"
mkdir -p "$REPORT_DIR/screenshots"
```

---

## 测试计划上下文

在回退到 git 差异启发式之前，检查更丰富的测试计划来源：

1. **项目范围的测试计划：** 检查 `~/.gstack/projects/` 中此仓库的最近 `*-test-plan-*.md` 文件
   ```bash
   eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
   ls -t ~/.gstack/projects/$SLUG/*-test-plan-*.md 2>/dev/null | head -1
   ```
2. **对话上下文：** 检查此对话中是否有先前的 `/plan-eng-review` 或 `/plan-ceo-review` 生成的测试计划输出
3. **使用更丰富的来源。** 仅当两者都不可用时才回退到 git 差异分析。

---

## 模式

### 差异感知（在功能分支上无 URL 时自动）

这是开发人员验证其工作的**主要模式**。当用户说 `/qa` 没有 URL 且仓库在功能分支上时，自动：

1. **分析分支差异** 以了解更改内容：
   ```bash
   git diff main...HEAD --name-only
   git log main..HEAD --oneline
   ```

2. **从更改的文件中识别受影响的页面/路由**：
   - 控制器/路由文件 → 它们服务的 URL 路径
   - 视图/模板/组件文件 → 渲染它们的页面
   - 模型/服务文件 → 使用这些模型的页面（检查引用它们的控制器）
   - CSS/样式文件 → 包含这些样式表的页面
   - API 端点 → 使用 `$B js "await fetch('/api/...')"` 直接测试它们
   - 静态页面（markdown，HTML）→ 直接导航到它们

3. **检测运行中的应用** — 检查常见的本地开发端口：
   ```bash
   $B goto http://localhost:3000 2>/dev/null && echo "Found app on :3000" || \
   $B goto http://localhost:4000 2>/dev/null && echo "Found app on :4000" || \
   $B goto http://localhost:8080 2>/dev/null && echo "Found app on :8080"
   ```
   如果未找到本地应用，检查 PR 或环境中的 staging/preview URL。如果都不行，向用户询问 URL。

4. **测试每个受影响的页面/路由：**
   - 导航到页面
   - 截图
   - 检查控制台是否有错误
   - 如果更改是交互式的（表单、按钮、流程），端到端测试交互
   - 使用 `snapshot -D` 在操作前后验证更改是否产生了预期效果

5. **交叉引用提交消息和 PR 描述** 以了解*意图* — 更改应该做什么？验证它实际上做到了。

6. **检查 TODOS.md**（如果存在）中与更改文件相关的已知错误或问题。如果 TODO 描述了此分支应该修复的错误，将其添加到测试计划中。如果在 QA 期间发现不在 TODOS.md 中的新错误，在报告中注明。

7. **报告发现** 范围限定在分支更改：
   - "测试的更改：此分支影响的 N 个页面/路由"
   - 每个：它工作吗？截图证据。
   - 相邻页面上有任何回归吗？

**如果用户在差异感知模式下提供 URL：** 使用该 URL 作为基础，但仍将测试范围限定在更改的文件。

### 完整（提供 URL 时的默认模式）
系统性探索。访问每个可达页面。记录 5-10 个有充分证据的问题。生成健康评分。根据应用大小，需要 5-15 分钟。

### 快速（`--quick`）
30 秒冒烟测试。访问主页 + 前 5 个导航目标。检查：页面加载？控制台错误？损坏的链接？生成健康评分。无详细问题文档。

### 回归（`--regression <baseline>`）
运行完整模式，然后加载先前运行的 `baseline.json`。差异：哪些问题已修复？哪些是新的？评分差异是多少？在报告中添加回归部分。

---

## 工作流

### 阶段 1：初始化

1. 找到浏览二进制文件（见上面的设置）
2. 创建输出目录
3. 将报告模板从 `qa/templates/qa-report-template.md` 复制到输出目录
4. 启动计时器以跟踪持续时间

### 阶段 2：认证（如果需要）

**如果用户指定了认证凭据：**

```bash
$B goto <login-url>
$B snapshot -i                    # 找到登录表单
$B fill @e3 "user@example.com"
$B fill @e4 "[REDACTED]"         # 永远不要在报告中包含真实密码
$B click @e5                      # 提交
$B snapshot -D                    # 验证登录成功
```

**如果用户提供了 cookie 文件：**

```bash
$B cookie-import cookies.json
$B goto <target-url>
```

**如果需要 2FA/OTP：** 向用户询问代码并等待。

**如果 CAPTCHA 阻止了你：** 告诉用户："请在浏览器中完成 CAPTCHA，然后告诉我继续。"

### 阶段 3：定向

获取应用程序的地图：

```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links                          # 映射导航结构
$B console --errors               # 登录时是否有错误？
```

**检测框架**（在报告元数据中注明）：
- HTML 中的 `__next` 或 `_next/data` 请求 → Next.js
- `csrf-token` 元标签 → Rails
- URL 中的 `wp-content` → WordPress
- 无页面刷新的客户端路由 → SPA

**对于 SPA：** `links` 命令可能返回很少的结果，因为导航是客户端的。使用 `snapshot -i` 来查找导航元素（按钮，菜单项）。

### 阶段 4：探索

系统地访问页面。在每个页面：

```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page-name.png"
$B console --errors
```

然后遵循**每页探索清单**（见 `qa/references/issue-taxonomy.md`）：

1. **视觉扫描** — 查看带注释的截图以查找布局问题
2. **交互元素** — 点击按钮、链接、控件。它们工作吗？
3. **表单** — 填写并提交。测试空、无效、边缘情况
4. **导航** — 检查所有进出路径
5. **状态** — 空状态、加载、错误、溢出
6. **控制台** — 交互后有任何新的 JS 错误？
7. **响应性** — 如果相关，检查移动视口：
   ```bash
   $B viewport 375x812
   $B screenshot "$REPORT_DIR/screenshots/page-mobile.png"
   $B viewport 1280x720
   ```

**深度判断：** 在核心功能（主页、仪表板、结账、搜索）上花费更多时间，在次要页面（关于、条款、隐私）上花费更少时间。

**快速模式：** 只访问主页 + 定向阶段的前 5 个导航目标。跳过每页清单 — 只检查：加载？控制台错误？可见的损坏链接？

### 阶段 5：记录

**发现每个问题后立即记录** — 不要批量处理。

**两个证据级别：**

**交互式错误**（损坏的流程，无效按钮，表单失败）：
1. 在操作前截图
2. 执行操作
3. 截图显示结果
4. 使用 `snapshot -D` 显示更改内容
5. 写入引用截图的重现步骤

```bash
$B screenshot "$REPORT_DIR/screenshots/issue-001-step-1.png"
$B click @e5
$B screenshot "$REPORT_DIR/screenshots/issue-001-result.png"
$B snapshot -D
```

**静态错误**（拼写错误，布局问题，缺失图像）：
1. 拍摄显示问题的单个带注释截图
2. 描述问题所在

```bash
$B snapshot -i -a -o "$REPORT_DIR/screenshots/issue-002.png"
```

**立即将每个问题写入报告** 使用 `qa/templates/qa-report-template.md` 中的模板格式。

### 阶段 6：结束

1. **计算健康评分** 使用下面的评分标准
2. **写入"前 3 个需要修复的问题"** — 3 个最高严重性的问题
3. **写入控制台健康摘要** — 汇总跨页面看到的所有控制台错误
4. **更新摘要表中的严重性计数**
5. **填写报告元数据** — 日期、持续时间、访问页面、截图计数、框架
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
- 在报告中添加回归部分

---

## 健康评分标准

计算每个类别得分（0-100），然后取加权平均值。

### 控制台（权重：15%）
- 0 个错误 → 100
- 1-3 个错误 → 70
- 4-10 个错误 → 40
- 10+ 个错误 → 10

### 链接（权重：10%）
- 0 个损坏 → 100
- 每个损坏的链接 → -15（最低 0）

### 每个类别评分（视觉、功能、用户体验、内容、性能、可访问性）
每个类别从 100 开始。每发现一个问题扣分：
- 关键问题 → -25
- 高问题 → -15
- 中问题 → -8
- 低问题 → -3
每个类别最低 0。

### 权重
| 类别 | 权重 |
|----------|--------|
| 控制台 | 15% |
| 链接 | 10% |
| 视觉 | 10% |
| 功能 | 20% |
| 用户体验 | 15% |
| 性能 | 10% |
| 内容 | 5% |
| 可访问性 | 15% |

### 最终评分
`score = Σ (category_score × weight)`

---

## 框架特定指南

### Next.js
- 检查控制台是否有水合错误（`Hydration failed`，`Text content did not match`）
- 监控网络中的 `_next/data` 请求 — 404 表示损坏的数据获取
- 测试客户端导航（点击链接，不要只是 `goto`）— 捕获路由问题
- 检查具有动态内容的页面上的 CLS（累积布局偏移）

### Rails
- 检查控制台中的 N+1 查询警告（如果在开发模式）
- 验证表单中存在 CSRF 令牌
- 测试 Turbo/Stimulus 集成 — 页面转换是否平滑？
- 检查 flash 消息是否正确显示和消失

### WordPress
- 检查插件冲突（来自不同插件的 JS 错误）
- 验证登录用户的管理栏可见性
- 测试 REST API 端点（`/wp-json/`）
- 检查混合内容警告（WP 常见）

### 通用 SPA（React，Vue，Angular）
- 使用 `snapshot -i` 进行导航 — `links` 命令错过客户端路由
- 检查过时状态（导航离开并返回 — 数据是否刷新？）
- 测试浏览器后退/前进 — 应用是否正确处理历史记录？
- 检查内存泄漏（长时间使用后监控控制台）

---

## 重要规则

1. **重现是一切。** 每个问题至少需要一张截图。无例外。
2. **记录前验证。** 重试问题一次以确认它是可重现的，不是偶然。
3. **永远不要包含凭据。** 在重现步骤中为密码写入 `[REDACTED]`。
4. **增量写入。** 发现问题时立即将每个问题附加到报告中。不要批量处理。
5. **永远不要阅读源代码。** 作为用户测试，不是开发人员。
6. **每次交互后检查控制台。** 视觉上不显示的 JS 错误仍然是错误。
7. **像用户一样测试。** 使用逼真的数据。端到端走完完整工作流程。
8. **深度优于广度。** 5-10 个有充分文档的问题，有证据 > 20 个模糊描述。
9. **永远不要删除输出文件。** 截图和报告累积 — 这是故意的。
10. **对棘手的 UI 使用 `snapshot -C`。** 找到可访问性树错过的可点击 div。
11. **向用户显示截图。** 每次 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 命令后，使用 Read 工具读取输出文件，以便用户可以内联看到它们。对于 `responsive`（3 个文件），读取所有三个。这很关键 — 否则，截图对用户是不可见的。

---

## 输出

将报告写入本地和项目范围的位置：

**本地：** `.gstack/qa-reports/qa-report-{domain}-{YYYY-MM-DD}.md`

**项目范围：** 为跨会话上下文写入测试结果工件：
```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mkdir -p ~/.gstack/projects/$SLUG
```
写入 `~/.gstack/projects/{slug}/{user}-{branch}-test-outcome-{datetime}.md`

### 输出结构

```
.gstack/qa-reports/
├── qa-report-{domain}-{YYYY-MM-DD}.md    # 结构化报告
├── screenshots/
│   ├── initial.png                        # 登录页面带注释截图
│   ├── issue-001-step-1.png               # 每个问题的证据
│   ├── issue-001-result.png
│   └── ...
└── baseline.json                          # 用于回归模式
```

报告文件名使用域名和日期：`qa-report-myapp-com-2026-03-12.md`

---

## 附加规则（仅 qa-only 特定）

11. **永远不要修复错误。** 只查找和记录。不要阅读源代码，编辑文件，或在报告中建议修复。你的工作是报告什么坏了，而不是修复它。使用 `/qa` 进行测试-修复-验证循环。
12. **未检测到测试框架？** 如果项目没有测试基础设施（没有测试配置文件，没有测试目录），在报告摘要中包含："未检测到测试框架。运行 `/qa` 来引导一个并启用回归测试生成。"