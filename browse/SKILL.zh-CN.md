---
name: browse
version: 1.1.0
description: |
  用于QA测试和站点内部使用的快速无头浏览器。导航任何URL，与元素交互，
  验证页面状态，比较操作前后的差异，拍摄带注释的截图，检查响应式布局，
  测试表单和上传，处理对话框，并断言元素状态。
  每个命令约100ms。当你需要测试功能、验证部署、内部测试用户流程或
  提供bug证据时使用。当被要求"在浏览器中打开"、"测试站点"、"截图"或"内部测试这个"时使用。
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

# browse：QA测试与内部使用

持久无头Chromium。首次调用自动启动（约3秒），然后每个命令约100ms。
状态在调用之间保持（cookie、标签页、登录会话）。

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

## 核心QA模式

### 1. 验证页面正确加载
```bash
$B goto https://yourapp.com
$B text                          # 内容加载？
$B console                       # JS错误？
$B network                       # 失败的请求？
$B is visible ".main-content"    # 关键元素存在？
```

### 2. 测试用户流程
```bash
$B goto https://app.com/login
$B snapshot -i                   # 查看所有交互元素
$B fill @e3 "user@test.com"
$B fill @e4 "password"
$B click @e5                     # 提交
$B snapshot -D                   # 差异：提交后发生了什么变化？
$B is visible ".dashboard"       # 成功状态存在？
```

### 3. 验证操作是否成功
```bash
$B snapshot                      # 基线
$B click @e3                     # 做某事
$B snapshot -D                   # 统一差异显示确切的变化
```

### 4. 错误报告的视觉证据
```bash
$B snapshot -i -a -o /tmp/annotated.png   # 带标签的截图
$B screenshot /tmp/bug.png                # 普通截图
$B console                                # 错误日志
```

### 5. 查找所有可点击元素（包括非ARIA）
```bash
$B snapshot -C                   # 查找带有cursor:pointer、onclick、tabindex的div
$B click @c1                     # 与它们交互
```

### 6. 断言元素状态
```bash
$B is visible ".modal"
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"
$B is checked "#agree-checkbox"
$B is editable "#name-field"
$B is focused "#search-input"
$B js "document.body.textContent.includes('Success')"
```

### 7. 测试响应式布局
```bash
$B responsive /tmp/layout        # 移动 + 平板 + 桌面截图
$B viewport 375x812              # 或设置特定视口
$B screenshot /tmp/mobile.png
```

### 8. 测试文件上传
```bash
$B upload "#file-input" /path/to/file.pdf
$B is visible ".upload-success"
```

### 9. 测试对话框
```bash
$B dialog-accept "yes"           # 设置处理程序
$B click "#delete-button"        # 触发对话框
$B dialog                        # 查看出现的内容
$B snapshot -D                   # 验证删除是否发生
```

### 10. 比较环境
```bash
$B diff https://staging.app.com https://prod.app.com
```

### 11. 向用户显示截图
在 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 之后，始终使用Read工具读取输出的PNG文件，以便用户可以看到它们。没有这个，截图是不可见的。

## 快照标志

快照是你理解和与页面交互的主要工具。

```
-i        --interactive           仅交互元素（按钮、链接、输入），带@e引用
-c        --compact               紧凑（无空结构节点）
-d <N>    --depth                 限制树深度（0 = 仅根，默认：无限制）
-s <sel>  --selector              限制到CSS选择器
-D        --diff                  与前一个快照的统一差异（首次调用存储基线）
-a        --annotate              带红色覆盖框和引用标签的注释截图
-o <path> --output                注释截图的输出路径（默认：/tmp/browse-annotated.png）
-C        --cursor-interactive    光标交互元素（@c引用 — 带有pointer、onclick的div）
```

所有标志可以自由组合。`-o` 仅在同时使用 `-a` 时适用。
示例：`$B snapshot -i -a -C -o /tmp/annotated.png`

**引用编号：** @e引用按树顺序顺序分配（@e1, @e2, ...）。
来自 `-C` 的@c引用单独编号（@c1, @c2, ...）。

快照后，在任何命令中使用@引用作为选择器：
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # 光标交互引用（来自-C）
```

**输出格式：** 带@引用ID的缩进可访问性树，每行一个元素。
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

导航时引用失效 — 在 `goto` 后再次运行 `snapshot`。

## 完整命令列表

### 导航
| 命令 | 描述 |
|------|------|
| `back` | 历史后退 |
| `forward` | 历史前进 |
| `goto <url>` | 导航到URL |
| `reload` | 重新加载页面 |
| `url` | 打印当前URL |

### 读取
| 命令 | 描述 |
|------|------|
| `accessibility` | 完整ARIA树 |
| `forms` | 表单字段（JSON格式） |
| `html [selector]` | 选择器的innerHTML（如果未找到则抛出），如果未给定选择器则返回完整页面HTML |
| `links` | 所有链接，格式为"文本 → href" |
| `text` | 清理后的页面文本 |

### 交互
| 命令 | 描述 |
|------|------|
| `click <sel>` | 点击元素 |
| `cookie <name>=<value>` | 在当前页面域上设置cookie |
| `cookie-import <json>` | 从JSON文件导入cookie |
| `cookie-import-browser [browser] [--domain d]` | 从Comet、Chrome、Arc、Brave或Edge导入cookie（打开选择器，或使用--domain直接导入） |
| `dialog-accept [text]` | 自动接受下一个alert/confirm/prompt。可选文本作为prompt响应发送 |
| `dialog-dismiss` | 自动关闭下一个对话框 |
| `fill <sel> <val>` | 填写输入 |
| `header <name>:<value>` | 设置自定义请求头（冒号分隔，敏感值自动脱敏） |
| `hover <sel>` | 悬停元素 |
| `press <key>` | 按键 — Enter, Tab, Escape, ArrowUp/Down/Left/Right, Backspace, Delete, Home, End, PageUp, PageDown，或修饰键如Shift+Enter |
| `scroll [sel]` | 将元素滚动到视图中，如果没有选择器则滚动到页面底部 |
| `select <sel> <val>` | 按值、标签或可见文本选择下拉选项 |
| `type <text>` | 输入到聚焦元素 |
| `upload <sel> <file> [file2...]` | 上传文件 |
| `useragent <string>` | 设置用户代理 |
| `viewport <WxH>` | 设置视口大小 |
| `wait <sel|--networkidle|--load>` | 等待元素、网络空闲或页面加载（超时：15秒） |

### 检查
| 命令 | 描述 |
|------|------|
| `attrs <sel|@ref>` | 元素属性（JSON格式） |
| `console [--clear|--errors]` | 控制台消息（--errors过滤为错误/警告） |
| `cookies` | 所有cookie（JSON格式） |
| `css <sel> <prop>` | 计算的CSS值 |
| `dialog [--clear]` | 对话框消息 |
| `eval <file>` | 从文件运行JavaScript并将结果作为字符串返回（路径必须在/tmp或cwd下） |
| `is <prop> <sel>` | 状态检查（visible/hidden/enabled/disabled/checked/editable/focused） |
| `js <expr>` | 运行JavaScript表达式并将结果作为字符串返回 |
| `network [--clear]` | 网络请求 |
| `perf` | 页面加载时间 |
| `storage [set k v]` | 读取所有localStorage + sessionStorage（JSON格式），或设置<key> <value>写入localStorage |

### 视觉
| 命令 | 描述 |
|------|------|
| `diff <url1> <url2>` | 页面之间的文本差异 |
| `pdf [path]` | 保存为PDF |
| `responsive [prefix]` | 在移动（375x812）、平板（768x1024）、桌面（1280x720）尺寸截图。保存为{prefix}-mobile.png等 |
| `screenshot [--viewport] [--clip x,y,w,h] [selector|@ref] [path]` | 保存截图（支持通过CSS/@ref裁剪元素，--clip区域，--viewport） |

### 快照
| 命令 | 描述 |
|------|------|
| `snapshot [flags]` | 带@e引用的可访问性树，用于元素选择。标志：-i仅交互，-c紧凑，-d N深度限制，-s sel范围，-D与之前差异，-a注释截图，-o路径输出，-C光标交互@c引用 |

### 元命令
| 命令 | 描述 |
|------|------|
| `chain` | 从JSON标准输入运行命令。格式：[["cmd","arg1",...],...]

### 标签页
| 命令 | 描述 |
|------|------|
| `closetab [id]` | 关闭标签页 |
| `newtab [url]` | 打开新标签页 |
| `tab <id>` | 切换到标签页 |
| `tabs` | 列出打开的标签页 |

### 服务器
| 命令 | 描述 |
|------|------|
| `restart` | 重启服务器 |
| `status` | 健康检查 |
| `stop` | 关闭服务器 |