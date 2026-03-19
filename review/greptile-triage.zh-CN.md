# Greptile 评论分类处理

用于在 GitHub PR 上获取、过滤和分类 Greptile 审查评论的共享参考。`/review`（步骤 2.5）和 `/ship`（步骤 3.75）都引用了本文档。

---

## 获取

运行以下命令检测 PR 并获取评论。两个 API 调用并行运行。

```bash
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner' 2>/dev/null)
PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null)
```

**如果任一命令失败或为空：** 静默跳过 Greptile 分类处理。此集成是附加的 — 工作流在没有它的情况下也能正常工作。

```bash
# 并行获取行级审查评论和顶级 PR 评论
gh api repos/$REPO/pulls/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | select(.position != null) | {id: .id, path: .path, line: .line, body: .body, html_url: .html_url, source: "line-level"}' > /tmp/greptile_line.json &
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | {id: .id, body: .body, html_url: .html_url, source: "top-level"}' > /tmp/greptile_top.json &
wait
```

**如果 API 出错或两个端点都没有 Greptile 评论：** 静默跳过。

行级评论上的 `position != null` 过滤器会自动跳过来自强制推送代码的过时评论。

---

## 抑制检查

派生项目特定的历史路径：
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
PROJECT_HISTORY="$HOME/.gstack/projects/$REMOTE_SLUG/greptile-history.md"
```

如果 `$PROJECT_HISTORY` 存在（每个项目的抑制记录），则读取它。每行记录先前的分类结果：

```
<date> | <repo> | <type:fp|fix|already-fixed> | <file-pattern> | <category>
```

**类别**（固定集合）：`race-condition`、`null-check`、`error-handling`、`style`、`type-safety`、`security`、`performance`、`correctness`、`other`

将每个获取的评论与以下条目不匹配：
- `type == fp`（仅抑制已知的误报，而不是先前已修复的真实问题）
- `repo` 与当前仓库匹配
- `file-pattern` 与评论的文件路径匹配
- `category` 与评论中的问题类型匹配

跳过匹配的评论作为 **已抑制**。

如果历史文件不存在或有无法解析的行，跳过这些行并继续 — 永远不要因格式错误的历史文件而失败。

---

## 分类

对于每个未抑制的评论：

1. **行级评论：** 读取指定 `path:line` 处的文件和周围上下文（±10 行）
2. **顶级评论：** 读取完整评论正文
3. 对照完整差异（`git diff origin/main`）和审查清单交叉引用评论
4. 分类：
   - **有效且可操作** — 当前代码中存在的真实错误、竞争条件、安全问题或正确性问题
   - **有效但已修复** — 在分支上的后续提交中已解决的真实问题。识别修复提交 SHA。
   - **误报** — 评论误解了代码，标记了其他地方已处理的内容，或属于风格噪音
   - **已抑制** — 已在上面的抑制检查中过滤

---

## 回复 API

回复 Greptile 评论时，根据评论来源使用正确的端点：

**行级评论**（来自 `pulls/$PR/comments`）：
```bash
gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies \
  -f body="<reply text>"
```

**顶级评论**（来自 `issues/$PR/comments`）：
```bash
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  -f body="<reply text>"
```

**如果回复 POST 失败**（例如，PR 已关闭，无写入权限）：警告并继续。不要因回复失败而停止工作流。

---

## 回复模板

对每个 Greptile 回复使用这些模板。始终包含具体证据 — 永远不要发布模糊的回复。

### 第 1 层（首次响应）— 友好，包含证据

**对于修复（用户选择修复问题）：**

```
**已修复** 在 `<commit-sha>`。

\`\`\`diff
- <旧的有问题的行>
+ <新的已修复的行>
\`\`\`

**原因：** <1 句话解释什么是错误的以及修复如何解决它>
```

**对于已修复（问题在分支上的先前提交中已解决）：**

```
**已修复** 在 `<commit-sha>`。

**已完成的工作：** <1-2 句话描述现有提交如何解决此问题>
```

**对于误报（评论不正确）：**

```
**不是错误。** <1 句话直接说明为什么这是不正确的>

**证据：**
- <显示模式安全/正确的特定代码引用>
- <例如，"nil 检查由 `ActiveRecord::FinderMethods#find` 处理，它会引发 RecordNotFound，而不是 nil">

**建议重新排名：** 这似乎是一个 `<style|noise|misread>` 问题，而不是 `<Greptile 所称的内容>`。考虑降低严重性。
```

### 第 2 层（Greptile 在先前回复后再次标记）— 坚定，证据充分

当升级检测（如下）识别到同一线程上存在先前的 GStack 回复时，使用第 2 层。包含最大证据以结束讨论。

```
**这已经过审查并确认为 [intentional/already-fixed/not-a-bug]。**

\`\`\`diff
<显示更改或安全模式的完整相关差异>
\`\`\`

**证据链：**
1. <显示安全模式或修复的 file:line 永久链接>
2. <解决问题的提交 SHA（如适用）>
3. <架构原理或设计决策（如适用）>

**建议重新排名：** 请重新校准 — 这是一个 `<实际类别>` 问题，不是 `<声称的类别>`。[如果有帮助，链接到特定文件更改的永久链接]
```

---

## 升级检测

在撰写回复之前，检查此评论线程是否已存在先前的 GStack 回复：

1. **对于行级评论：** 通过 `gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies` 获取回复。检查是否有任何回复正文包含 GStack 标记：`**Fixed**`、`**Not a bug.**`、`**Already fixed**`。

2. **对于顶级评论：** 扫描获取的问题评论，查找在 Greptile 评论之后发布且包含 GStack 标记的回复。

3. **如果存在先前的 GStack 回复 AND Greptile 在同一文件+类别上再次发布：** 使用第 2 层（坚定）模板。

4. **如果不存在先前的 GStack 回复：** 使用第 1 层（友好）模板。

如果升级检测失败（API 错误，线程不明确）：默认为第 1 层。永远不要在不明确的情况下升级。

---

## 严重性评估和重新排名

在对评论进行分类时，还要评估 Greptile 的隐含严重性是否与现实相符：

- 如果 Greptile 将某事物标记为 **security/correctness/race-condition** 问题，但实际上是 **style/performance** 小问题：在回复中包含 `**建议重新排名：**` 请求更正类别。
- 如果 Greptile 将低严重性的样式问题标记为关键问题：在回复中反驳。
- 始终具体说明为什么重新排名是合理的 — 引用代码和行号，而不是意见。

---

## 历史文件写入

在写入之前，确保两个目录都存在：
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
mkdir -p "$HOME/.gstack/projects/$REMOTE_SLUG"
mkdir -p ~/.gstack
```

将每个分类结果附加一行到 **两个** 文件（每个项目用于抑制，全局用于回顾）：
- `~/.gstack/projects/$REMOTE_SLUG/greptile-history.md`（每个项目）
- `~/.gstack/greptile-history.md`（全局汇总）

格式：
```
<YYYY-MM-DD> | <owner/repo> | <type> | <file-pattern> | <category>
```

示例条目：
```
2026-03-13 | garrytan/myapp | fp | app/services/auth_service.rb | race-condition
2026-03-13 | garrytan/myapp | fix | app/models/user.rb | null-check
2026-03-13 | garrytan/myapp | already-fixed | lib/payments.rb | error-handling
```

---

## 输出格式

在输出标题中包含 Greptile 摘要：
```
+ N Greptile 评论（X 有效，Y 已修复，Z 误报）
```

对于每个分类的评论，显示：
- 分类标签：`[VALID]`、`[FIXED]`、`[FALSE POSITIVE]`、`[SUPPRESSED]`
- 文件：行引用（对于行级）或 `[top-level]`（对于顶级）
- 一行正文摘要
- 永久链接 URL（`html_url` 字段）