---
name: setup-browser-cookies
version: 1.0.0
description: |
  从您的真实浏览器（Comet、Chrome、Arc、Brave、Edge）导入 Cookie 到
  无头浏览会话中。打开一个交互式选择器 UI，您可以在其中选择要导入的
  Cookie 域名。在 QA 测试已认证页面之前使用。当被要求
  "导入 Cookie"、"登录网站"或"验证浏览器"时使用。
allowed-tools:
  - Bash
  - Read
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

# 设置浏览器 Cookie

从您的真实 Chromium 浏览器导入已登录会话到无头浏览会话中。

## 工作原理

1. 找到浏览二进制文件
2. 运行 `cookie-import-browser` 检测已安装的浏览器并打开选择器 UI
3. 用户在浏览器中选择要导入的 Cookie 域名
4. Cookie 被解密并加载到 Playwright 会话中

## 步骤

### 1. 找到浏览二进制文件

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

### 2. 打开 Cookie 选择器

```bash
$B cookie-import-browser
```

这会自动检测已安装的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge）并在您的默认浏览器中打开
一个交互式选择器 UI，您可以：
- 在已安装的浏览器之间切换
- 搜索域名
- 点击 "+" 导入域名的 Cookie
- 点击垃圾桶移除已导入的 Cookie

告诉用户：**"Cookie 选择器已打开 — 在浏览器中选择要导入的域名，然后告诉我您已完成。"**

### 3. 直接导入（替代方法）

如果用户直接指定域名（例如 `/setup-browser-cookies github.com`），跳过 UI：

```bash
$B cookie-import-browser comet --domain github.com
```

如果指定了浏览器，将 `comet` 替换为适当的浏览器。

### 4. 验证

用户确认完成后：

```bash
$B cookies
```

向用户显示已导入 Cookie 的摘要（域名计数）。

## 注意事项

- 每个浏览器的首次导入可能会触发 macOS 钥匙串对话框 — 点击"允许"/"始终允许"
- Cookie 选择器与浏览服务器在同一端口上提供服务（无额外进程）
- UI 中只显示域名和 Cookie 计数 — 不暴露 Cookie 值
- 浏览会话在命令之间保持 Cookie，因此导入的 Cookie 立即生效