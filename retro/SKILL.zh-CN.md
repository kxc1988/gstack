---
name: retro
version: 2.0.0
description: |
  每周工程回顾。分析提交历史、工作模式和代码质量指标，
  具有持久历史和趋势跟踪。团队感知：按个人贡献分解，
  包含表扬和成长领域。当被要求"每周回顾"、"我们交付了什么"或"工程回顾"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
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

## 检测默认分支

在收集数据之前，检测仓库的默认分支名称：
`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

如果失败，回退到 `main`。在下面的说明中说 `origin/<default>` 的地方使用检测到的名称。

---

# /retro — 每周工程回顾

生成全面的工程回顾，分析提交历史、工作模式和代码质量指标。团队感知：识别运行命令的用户，然后分析每个贡献者，包括个人表扬和成长机会。专为使用 Claude Code 作为力量倍增器的高级 IC/CTO 级构建者设计。

## 用户可调用
当用户输入 `/retro` 时，运行此技能。

## 参数
- `/retro` — 默认：最近 7 天
- `/retro 24h` — 最近 24 小时
- `/retro 14d` — 最近 14 天
- `/retro 30d` — 最近 30 天
- `/retro compare` — 比较当前窗口与先前相同长度的窗口
- `/retro compare 14d` — 比较具有明确窗口

## 说明

解析参数以确定时间窗口。如果未给出参数，默认为 7 天。对于 git log 查询，使用 `--since="N days ago"`、`--since="N hours ago"` 或 `--since="N weeks ago"`（对于 `w` 单位）。所有时间应在 **太平洋时间** 报告（转换时间戳时使用 `TZ=America/Los_Angeles`）。

**参数验证：** 如果参数与数字后跟 `d`、`h` 或 `w`、单词 `compare` 或 `compare` 后跟数字和 `d`/`h`/`w` 不匹配，显示此用法并停止：
```
用法：/retro [窗口]
  /retro              — 最近 7 天（默认）
  /retro 24h          — 最近 24 小时
  /retro 14d          — 最近 14 天
  /retro 30d          — 最近 30 天
  /retro compare      — 比较此期间与先前期间
  /retro compare 14d  — 比较具有明确窗口
```

### 步骤 1：收集原始数据

首先，获取 origin 并识别当前用户：
```bash
git fetch origin <default> --quiet
# 识别谁在运行回顾
git config user.name
git config user.email
```

`git config user.name` 返回的名称是 **"你"** — 阅读此回顾的人。所有其他作者都是队友。使用此来定向叙述："你的"提交 vs 队友贡献。

并行运行所有这些 git 命令（它们是独立的）：

```bash
# 1. 窗口中的所有提交，带时间戳、主题、哈希、作者、更改的文件、插入、删除
git log origin/<default> --since="<window>" --format="%H|%aN|%ae|%ai|%s" --shortstat

# 2. 每个提交的测试 vs 总 LOC 分解，带作者
#    每个提交块以 COMMIT:<hash>|<author> 开始，后跟 numstat 行。
#    将测试文件（匹配 test/|spec/|__tests__/）与生产文件分开。
git log origin/<default> --since="<window>" --format="COMMIT:%H|%aN" --numstat

# 3. 用于会话检测和小时分布的提交时间戳（带作者）
#    使用 TZ=America/Los_Angeles 进行太平洋时间转换
TZ=America/Los_Angeles git log origin/<default> --since="<window>" --format="%at|%aN|%ai|%s" | sort -n

# 4. 最频繁更改的文件（热点分析）
git log origin/<default> --since="<window>" --format="" --name-only | grep -v '^$' | sort | uniq -c | sort -rn

# 5. 从提交消息中提取 PR 编号（提取 #NNN 模式）
git log origin/<default> --since="<window>" --format="%s" | grep -oE '#[0-9]+' | sed 's/^#//' | sort -n | uniq | sed 's/^/#/'

# 6. 每个作者的文件热点（谁触摸什么）
git log origin/<default> --since="<window>" --format="AUTHOR:%aN" --name-only

# 7. 每个作者的提交计数（快速摘要）
git shortlog origin/<default> --since="<window>" -sn --no-merges

# 8. Greptile 分类历史（如果可用）
cat ~/.gstack/greptile-history.md 2>/dev/null || true

# 9. TODOS.md 待办事项（如果可用）
cat TODOS.md 2>/dev/null || true

# 10. 测试文件计数
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' 2>/dev/null | grep -v node_modules | wc -l

# 11. 窗口中的回归测试提交
git log origin/<default> --since="<window>" --oneline --grep="test(qa):" --grep="test(design):" --grep="test: coverage"

# 12. 窗口中更改的测试文件
git log origin/<default> --since="<window>" --format="" --name-only | grep -E '\.(test|spec)\.' | sort -u | wc -l
```

### 步骤 2：计算指标

计算并在摘要表中呈现这些指标：

| 指标 | 值 |
|--------|-------|
| 提交到 main | N |
| 贡献者 | N |
| 合并的 PR | N |
| 总插入 | N |
| 总删除 | N |
| 净 LOC 添加 | N |
| 测试 LOC（插入） | N |
| 测试 LOC 比率 | N% |
| 版本范围 | vX.Y.Z.W → vX.Y.Z.W |
| 活动天数 | N |
| 检测到的会话 | N |
| 平均 LOC/会话小时 | N |
| Greptile 信号 | N%（Y 捕获，Z 误报） |
| 测试健康 | N 总测试 · M 本期添加 · K 回归测试 |

然后立即在下方显示 **每个作者的排行榜**：

```
贡献者         提交   +/-          主要领域
你（garry）              32   +2400/-300   browse/
alice                    12   +800/-150    app/services/
bob                       3   +120/-40     tests/
```

按提交降序排序。当前用户（来自 `git config user.name`）始终首先出现，标记为"你（姓名）"。

**Greptile 信号（如果历史存在）：** 阅读 `~/.gstack/greptile-history.md`（在步骤 1，命令 8 中获取）。按日期过滤回顾时间窗口内的条目。按类型计数条目：`fix`、`fp`、`already-fixed`。计算信号比率：`(fix + already-fixed) / (fix + already-fixed + fp)`。如果窗口中没有条目或文件不存在，跳过 Greptile 指标行。静默跳过无法解析的行。

**待办事项健康（如果 TODOS.md 存在）：** 阅读 `TODOS.md`（在步骤 1，命令 9 中获取）。计算：
- 总开放 TODO（排除 `## 已完成` 部分中的项目）
- P0/P1 计数（关键/紧急项目）
- P2 计数（重要项目）
- 本期完成的项目（完成部分中日期在回顾窗口内的项目）
- 本期添加的项目（交叉引用窗口内修改 TODOS.md 的提交的 git log）

在指标表中包含：
```
| 待办事项健康 | N 开放（X P0/P1，Y P2）· Z 本期完成 |
```

如果 TODOS.md 不存在，跳过待办事项健康行。

### 步骤 3：提交时间分布

使用条形图显示太平洋时间的每小时直方图：

```
小时  提交  ████████████████
 00:    4      ████
 07:    5      █████
 ...
```

识别并指出：
- 高峰小时
- 死亡区域
- 模式是双峰（上午/晚上）还是连续
- 深夜编码集群（晚上 10 点后）

### 步骤 4：工作会话检测

使用 **45 分钟间隔** 阈值检测会话。对于每个会话报告：
- 开始/结束时间（太平洋时间）
- 提交数量
- 持续时间（分钟）

分类会话：
- **深度会话**（50+ 分钟）
- **中等会话**（20-50 分钟）
- **微会话**（<20 分钟，通常是单次提交的即发即忘）

计算：
- 总活跃编码时间（会话持续时间之和）
- 平均会话长度
- 每小时活跃时间的 LOC

### 步骤 5：提交类型分解

按常规提交前缀（feat/fix/refactor/test/chore/docs）分类。显示为百分比条：

```
feat:     20  (40%)  ████████████████████
fix:      27  (54%)  ███████████████████████████
refactor:  2  ( 4%)  ██
```

如果修复比率超过 50%，标记 — 这表明"快速发布，快速修复"模式，可能表明审查差距。

### 步骤 6：热点分析

显示前 10 个最常更改的文件。标记：
- 更改 5+ 次的文件（ churn 热点）
- 热点列表中的测试文件 vs 生产文件
- VERSION/CHANGELOG 频率（版本纪律指标）

### 步骤 7：PR 大小分布

从提交差异估计 PR 大小并对其进行分类：
- **小**（<100 LOC）
- **中等**（100-500 LOC）
- **大**（500-1500 LOC）
- **XL**（1500+ LOC）— 用文件计数标记这些

### 步骤 8：专注度分数 + 本周最佳发布

**专注度分数：** 计算触及单个最常更改的顶级目录（例如 `app/services/`、`app/views/`）的提交百分比。分数越高 = 更深入的专注工作。分数越低 = 分散的上下文切换。报告为："专注度分数：62%（app/services/）"

**本周最佳发布：** 自动识别窗口中 LOC 最高的单个 PR。突出显示：
- PR 编号和标题
- 更改的 LOC
- 为什么重要（从提交消息和触及的文件推断）

### 步骤 9：团队成员分析

对于每个贡献者（包括当前用户），计算：

1. **提交和 LOC** — 总提交、插入、删除、净 LOC
2. **关注领域** — 他们最常触及的目录/文件（前 3 个）
3. **提交类型混合** — 他们个人的 feat/fix/refactor/test 分解
4. **会话模式** — 他们何时编码（高峰小时）、会话计数
5. **测试纪律** — 他们个人的测试 LOC 比率
6. **最大发布** — 窗口中他们单个最高影响的提交或 PR

**对于当前用户（"你"）：** 此部分得到最深入的处理。包括来自单独回顾的所有细节 — 会话分析、时间模式、专注度分数。以第一人称框架："你的高峰小时..."、"你的最大发布..."

**对于每个队友：** 写 2-3 句话，涵盖他们的工作内容和模式。然后：

- **表扬**（1-2 具体事项）：锚定在实际提交中。不是"很棒的工作" — 确切说明什么是好的。例如："在 3 个专注会话中完成了整个认证中间件重写，测试覆盖率 45%"、"每个 PR 都在 200 LOC 以下 — 纪律性分解。"
- **成长机会**（1 具体事项）：作为提升建议，而非批评。锚定在实际数据中。例如："本周测试比率为 12% — 在支付模块变得更复杂之前添加测试覆盖率会有回报"、"同一文件上的 5 个修复提交表明原始 PR 可能需要审查。"

**如果只有一个贡献者（单独仓库）：** 跳过团队分解，继续按之前的方式进行 — 回顾是个人的。

**如果有 Co-Authored-By 尾部：** 解析提交消息中的 `Co-Authored-By:` 行。将这些作者与主要作者一起归功于提交。注意 AI 共同作者（例如 `noreply@anthropic.com`）但不要将他们作为团队成员包括 — 而是将"AI 辅助提交"作为单独的指标跟踪。

### 步骤 10：周环比趋势（如果窗口 >= 14 天）

如果时间窗口为 14 天或更长，拆分为周桶并显示趋势：
- 每周提交（总数和每个作者）
- 每周 LOC
- 每周测试比率
- 每周修复比率
- 每周会话计数

### 步骤 11：连续跟踪

计算从今天开始，连续向 origin/<default> 提交至少 1 次的天数。同时跟踪团队连续和个人连续：

```bash
# 团队连续：所有唯一提交日期（太平洋时间）— 无硬截止
TZ=America/Los_Angeles git log origin/<default> --format="%ad" --date=format:"%Y-%m-%d" | sort -u

# 个人连续：仅当前用户的提交
TZ=America/Los_Angeles git log origin/<default> --author="<user_name>" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

从今天开始向后计数 — 连续多少天至少有一次提交？此查询完整历史，因此任何长度的连续都被准确报告。显示两者：
- "团队发布连续：47 天"
- "你的发布连续：32 天"

### 步骤 12：加载历史并比较

在保存新快照之前，检查之前的回顾历史：

```bash
ls -t .context/retros/*.json 2>/dev/null
```

**如果存在之前的回顾：** 使用 Read 工具加载最近的一次。计算关键指标的增量并包含 **与上次回顾的趋势** 部分：
```
                    上次        现在         增量
测试比率:         22%    →    41%         ↑19pp
会话:           10     →    14          ↑4
LOC/小时:           200    →    350         ↑75%
修复比率:          54%    →    30%         ↓24pp (改善)
提交:            32     →    47          ↑47%
深度会话:      3      →    5           ↑2
```

**如果没有之前的回顾：** 跳过比较部分并附加："记录的第一次回顾 — 下周再次运行以查看趋势。"

### 步骤 13：保存回顾历史

计算所有指标（包括连续）并加载任何之前的历史进行比较后，保存 JSON 快照：

```bash
mkdir -p .context/retros
```

确定今天的下一个序列号（用实际日期替换 `$(date +%Y-%m-%d)`）：
```bash
# 计算今天现有的回顾以获取下一个序列号
today=$(TZ=America/Los_Angeles date +%Y-%m-%d)
existing=$(ls .context/retros/${today}-*.json 2>/dev/null | wc -l | tr -d ' ')
next=$((existing + 1))
# 保存为 .context/retros/${today}-${next}.json
```

使用 Write 工具保存具有此架构的 JSON 文件：
```json
{
  "date": "2026-03-08",
  "window": "7d",
  "metrics": {
    "commits": 47,
    "contributors": 3,
    "prs_merged": 12,
    "insertions": 3200,
    "deletions": 800,
    "net_loc": 2400,
    "test_loc": 1300,
    "test_ratio": 0.41,
    "active_days": 6,
    "sessions": 14,
    "deep_sessions": 5,
    "avg_session_minutes": 42,
    "loc_per_session_hour": 350,
    "feat_pct": 0.40,
    "fix_pct": 0.30,
    "peak_hour": 22,
    "ai_assisted_commits": 32
  },
  "authors": {
    "Garry Tan": { "commits": 32, "insertions": 2400, "deletions": 300, "test_ratio": 0.41, "top_area": "browse/" },
    "Alice": { "commits": 12, "insertions": 800, "deletions": 150, "test_ratio": 0.35, "top_area": "app/services/" }
  },
  "version_range": ["1.16.0.0", "1.16.1.0"],
  "streak_days": 47,
  "tweetable": "Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm",
  "greptile": {
    "fixes": 3,
    "fps": 1,
    "already_fixed": 2,
    "signal_pct": 83
  }
}
```

**注意：** 仅当 `~/.gstack/greptile-history.md` 存在且在时间窗口内有条目时才包含 `greptile` 字段。仅当 `TODOS.md` 存在时才包含 `backlog` 字段。仅当找到测试文件（命令 10 返回 > 0）时才包含 `test_health` 字段。如果任何没有数据，完全省略该字段。

当测试文件存在时，在 JSON 中包含测试健康数据：
```json
  "test_health": {
    "total_test_files": 47,
    "tests_added_this_period": 5,
    "regression_test_commits": 3,
    "test_files_changed": 8
  }
```

当 TODOS.md 存在时，在 JSON 中包含待办事项数据：
```json
  "backlog": {
    "total_open": 28,
    "p0_p1": 2,
    "p2": 8,
    "completed_this_period": 3,
    "added_this_period": 1
  }
```

### 步骤 14：编写叙述

将输出结构化为：

---

**可推文摘要**（第一行，在所有内容之前）：
```
Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm | Streak: 47d
```

## 工程回顾：[日期范围]

### 摘要表
（来自步骤 2）

### 与上次回顾的趋势
（来自步骤 11，在保存前加载 — 如果是第一次回顾则跳过）

### 时间和会话模式
（来自步骤 3-4）

解释团队范围模式含义的叙述：
- 最高效的时间是什么时候以及什么驱动它们
- 会话是否随着时间变长或变短
- 每天活跃编码的估计小时数（团队汇总）
- 值得注意的模式：团队成员是同时编码还是轮班？

### 发布速度
（来自步骤 5-7）

叙述涵盖：
- 提交类型混合及其揭示的内容
- PR 大小纪律（PR 是否保持小？）
- 修复链检测（同一子系统上的修复提交序列）
- 版本升级纪律

### 代码质量信号
- 测试 LOC 比率趋势
- 热点分析（相同文件是否在 churn？）
- 任何应该拆分的 XL PR
- Greptile 信号比率和趋势（如果历史存在）："Greptile：X% 信号（Y 有效捕获，Z 误报）"

### 测试健康
- 总测试文件：N（来自命令 10）
- 本期添加的测试：M（来自命令 12 — 更改的测试文件）
- 回归测试提交：列出来自命令 11 的 `test(qa):` 和 `test(design):` 和 `test: coverage` 提交
- 如果存在之前的回顾并具有 `test_health`：显示增量 "测试计数：{last} → {now} (+{delta})"
- 如果测试比率 < 20%：标记为成长领域 — "100% 测试覆盖率是目标。测试使直觉编码安全。"

### 专注度和亮点
（来自步骤 8）
- 专注度分数及解释
- 本周最佳发布标注

### 你的一周（个人深入分析）
（来自步骤 9，仅当前用户）

这是用户最关心的部分。包括：
- 他们的个人提交计数、LOC、测试比率
- 他们的会话模式和高峰小时
- 他们的关注领域
- 他们的最大发布
- **你做得好的地方**（2-3 具体事项，锚定在提交中）
- **提升的地方**（1-2 具体、可操作的建议）

### 团队分解
（来自步骤 9，每个队友 — 如果是单独仓库则跳过）

对于每个队友（按提交降序排序），写一个部分：

#### [姓名]
- **他们发布的内容**：2-3 句话关于他们的贡献、关注领域和提交模式
- **表扬**：1-2 具体他们做得好的事情，锚定在实际提交中。要真诚 — 你在 1:1 中实际会说什么？例如：
  - "在 3 个小型、可审查的 PR 中清理了整个认证模块 — 教科书式分解"
  - "为每个新端点添加了集成测试，不仅仅是愉快路径"
  - "修复了导致仪表板 2 秒加载时间的 N+1 查询"
- **成长机会**：1 具体、建设性的建议。作为投资，而非批评。例如：
  - "支付模块的测试覆盖率为 8% — 在下一个功能叠加之前值得投资"
  - "5 个 PR 中有 3 个是 800+ LOC — 拆分这些会更早发现问题并使审查更容易"
  - "所有提交都在凌晨 1-4 点之间 — 可持续的节奏对长期代码质量很重要"

**AI 协作说明：** 如果许多提交有 `Co-Authored-By` AI 尾部（例如 Claude、Copilot），将 AI 辅助提交百分比作为团队指标。中性框架 — "N% 的提交是 AI 辅助的" — 无判断。

### 团队三大胜利
识别窗口中整个团队发布的 3 个最高影响的事情。每个：
- 它是什么
- 谁发布了它
- 为什么重要（产品/架构影响）

### 3 个需要改进的地方
具体、可操作、锚定在实际提交中。混合个人和团队级建议。措辞为"为了变得更好，团队可以..."

### 下周的 3 个习惯
小、实用、现实。每个必须是需要 <5 分钟采用的东西。至少一个应该是团队导向的（例如，"当天互相审查彼此的 PR"）。

### 周环比趋势
（如适用，来自步骤 10）

---

## 比较模式

当用户运行 `/retro compare`（或 `/retro compare 14d`）时：

1. 使用 `--since="7 days ago"` 计算当前窗口（默认 7 天）的指标
2. 使用 `--since` 和 `--until` 计算前一个相同长度窗口的指标，以避免重叠（例如，对于 7 天窗口，`--since="14 days ago" --until="7 days ago"`）
3. 显示带有增量和箭头的并排比较表
4. 写一个简短的叙述，突出最大的改进和倒退
5. 仅将当前窗口快照保存到 `.context/retros/`（与正常回顾运行相同）；**不要** 持久化前一个窗口的指标。

## 语气

- 鼓励但坦率，不娇惯
- 具体和具体 — 始终锚定在实际提交/代码中
- 跳过通用表扬（"做得好！"）— 确切说明什么是好的以及为什么
- 将改进框架为提升，而非批评
- **表扬应该感觉像你在 1:1 中实际会说的话** — 具体、应得、真诚
- **成长建议应该感觉像投资建议** — "这值得你的时间因为..." 不是 "你在...上失败了"
- 永远不要负面比较队友。每个人的部分独立存在。
- 保持总输出约 3000-4500 字（稍微长一点以适应团队部分）
- 使用 markdown 表格和代码块进行数据，使用散文进行叙述
- 直接输出到对话 — 不要写入文件系统（除了 `.context/retros/` JSON 快照）

## 重要规则

- 所有叙述输出直接在对话中给用户。唯一写入的文件是 `.context/retros/` JSON 快照。
- 对所有 git 查询使用 `origin/<default>`（不是可能过时的本地 main）
- 将所有时间戳转换为太平洋时间显示（使用 `TZ=America/Los_Angeles`）
- 如果窗口没有提交，说明并建议不同的窗口
- 将 LOC/小时四舍五入到最接近的 50
- 将合并提交视为 PR 边界
- 不要阅读 CLAUDE.md 或其他文档 — 此技能是自包含的
- 首次运行（无先前回顾）时，优雅地跳过比较部分