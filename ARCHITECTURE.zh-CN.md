# 架构

本文档解释 **为什么** gstack 是这样构建的。有关设置和命令，请参阅 CLAUDE.md。有关贡献，请参阅 CONTRIBUTING.md。

## 核心思想

gstack 为 Claude 代码提供了一个持久的浏览器和一套固执己见的工作流技能。浏览器是困难的部分——其他一切都是 Markdown。

关键洞察：与浏览器交互的 AI 代理需要 **亚秒级延迟** 和 **持久状态**。如果每个命令都冷启动浏览器，你会为每个工具调用等待 3-5 秒。如果浏览器在命令之间死亡，你会丢失 cookie、标签页和登录会话。因此，gstack 运行一个长期存在的 Chromium 守护进程，CLI 通过 localhost HTTP 与之通信。

```
Claude 代码                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  工具调用: $B snapshot -i    │  CLI (编译二进制文件)│
  ─────────────────────────→   │  • 读取状态文件      │
                               │  • POST /command     │
                               │    到 localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  服务器 (Bun.serve)   │
                               │  • 分发命令           │
                               │  • 与 Chromium 通信  │
                               │  • 返回纯文本         │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (无头)      │
                               │  • 持久标签页         │
                               │  • cookie 保留        │
                               │  • 30分钟空闲超时     │
                               └───────────────────────┘
```

第一次调用启动所有内容（约 3 秒）。之后的每次调用：约 100-200ms。

## 为什么选择 Bun

Node.js 也可以工作。Bun 在这里更好，有三个原因：

1. **编译二进制文件**。`bun build --compile` 生成一个单一的 ~58MB 可执行文件。运行时没有 `node_modules`，没有 `npx`，没有 PATH 配置。二进制文件直接运行。这很重要，因为 gstack 安装到 `~/.claude/skills/`，用户不期望在那里管理 Node.js 项目。

2. **原生 SQLite**。Cookie 解密直接读取 Chromium 的 SQLite cookie 数据库。Bun 内置了 `new Database()`——没有 `better-sqlite3`，没有原生插件编译，没有 gyp。在不同机器上少了一件可能出错的事情。

3. **原生 TypeScript**。服务器在开发期间以 `bun run server.ts` 运行。没有编译步骤，没有 `ts-node`，没有需要调试的源映射。编译后的二进制文件用于部署；源文件用于开发。

4. **内置 HTTP 服务器**。`Bun.serve()` 快速、简单，不需要 Express 或 Fastify。服务器总共处理约 10 个路由。框架会是多余的。

瓶颈始终是 Chromium，而不是 CLI 或服务器。Bun 的启动速度（编译二进制文件约 1ms，而 Node 约 100ms）很好但不是我们选择它的原因。编译二进制文件和原生 SQLite 才是。

## 守护进程模型

### 为什么不为每个命令启动一个浏览器？

Playwright 可以在约 2-3 秒内启动 Chromium。对于单个截图，这没问题。对于有 20+ 命令的 QA 会话，这是 40+ 秒的浏览器启动开销。更糟的是：你在命令之间丢失所有状态。Cookie、localStorage、登录会话、打开的标签页——全部消失。

守护进程模型意味着：

- **持久状态**。登录一次，保持登录状态。打开一个标签页，它保持打开状态。localStorage 在命令之间持久化。
- **亚秒级命令**。第一次调用后，每个命令只是一个 HTTP POST。约 100-200ms 往返，包括 Chromium 的工作。
- **自动生命周期**。服务器在首次使用时自动启动，在 30 分钟空闲后自动关闭。不需要进程管理。

### 状态文件

服务器写入 `.gstack/browse.json`（通过 tmp + 重命名的原子写入，模式 0o600）：

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLI 读取此文件以找到服务器。如果文件缺失、过时或 PID 已死，CLI 会生成一个新服务器。

### 端口选择

10000-60000 之间的随机端口（冲突时最多重试 5 次）。这意味着 10 个 Conductor 工作区可以各自运行自己的浏览守护进程，零配置且零端口冲突。旧方法（扫描 9400-9409）在多工作区设置中不断失败。

### 版本自动重启

构建将 `git rev-parse HEAD` 写入 `browse/dist/.version`。在每次 CLI 调用时，如果二进制版本与运行中服务器的 `binaryVersion` 不匹配，CLI 会杀死旧服务器并启动一个新服务器。这完全防止了"过时二进制文件"类的错误——重建二进制文件，下一个命令自动使用它。

## 安全模型

### 仅本地主机

HTTP 服务器绑定到 `localhost`，而不是 `0.0.0.0`。它无法从网络访问。

### 承载令牌认证

每个服务器会话生成一个随机 UUID 令牌，写入模式为 0o600（仅所有者可读）的状态文件。每个 HTTP 请求必须包含 `Authorization: Bearer <token>`。如果令牌不匹配，服务器返回 401。

这可以防止同一机器上的其他进程与你的浏览服务器通信。Cookie 选择器 UI (`/cookie-picker`) 和健康检查 (`/health`) 是例外——它们仅本地主机且不执行命令。

### Cookie 安全

Cookie 是 gstack 处理的最敏感数据。设计：

1. **钥匙串访问需要用户批准**。每个浏览器的第一次 cookie 导入都会触发 macOS 钥匙串对话框。用户必须点击"允许"或"始终允许"。gstack 永远不会静默访问凭据。

2. **解密在进程内发生**。Cookie 值在内存中解密（PBKDF2 + AES-128-CBC），加载到 Playwright 上下文中，永远不会以明文形式写入磁盘。Cookie 选择器 UI 永远不会显示 cookie 值——只显示域名和计数。

3. **数据库是只读的**。gstack 将 Chromium cookie DB 复制到临时文件（以避免与运行中的浏览器的 SQLite 锁定冲突）并以只读方式打开。它永远不会修改你真实浏览器的 cookie 数据库。

4. **密钥缓存是按会话的**。钥匙串密码 + 派生的 AES 密钥在服务器生命周期内缓存在内存中。当服务器关闭（空闲超时或显式停止）时，缓存消失。

5. **日志中没有 cookie 值**。控制台、网络和对话框日志永远不包含 cookie 值。`cookies` 命令输出 cookie 元数据（域名、名称、过期时间）但值被截断。

### Shell 注入防护

浏览器注册表（Comet、Chrome、Arc、Brave、Edge）是硬编码的。数据库路径由已知常量构造，从不来自用户输入。钥匙串访问使用 `Bun.spawn()` 与显式参数数组，而不是 shell 字符串插值。

## 引用系统

引用（`@e1`、`@e2`、`@c1`）是代理在不编写 CSS 选择器或 XPath 的情况下寻址页面元素的方式。

### 工作原理

```
1. 代理运行: $B snapshot -i
2. 服务器调用 Playwright 的 page.accessibility.snapshot()
3. 解析器遍历 ARIA 树，分配顺序引用: @e1, @e2, @e3...
4. 对于每个引用，构建 Playwright Locator: getByRole(role, { name }).nth(index)
5. 在 BrowserManager 实例上存储 Map<string, RefEntry>（role + name + Locator）
6. 将带注释的树作为纯文本返回

稍后:
7. 代理运行: $B click @e3
8. 服务器解析 @e3 → Locator → locator.click()
```

### 为什么使用 Locator，而不是 DOM 突变

明显的方法是向 DOM 注入 `data-ref="@e1"` 属性。这在以下情况下会失败：

- **CSP（内容安全策略）**。许多生产站点阻止来自脚本的 DOM 修改。
- **React/Vue/Svelte 水合**。框架协调可以剥离注入的属性。
- **Shadow DOM**。无法从外部访问阴影根内部。

Playwright Locator 是 DOM 外部的。它们使用可访问性树（Chromium 在内部维护）和 `getByRole()` 查询。没有 DOM 突变，没有 CSP 问题，没有框架冲突。

### 引用生命周期

引用在导航时清除（主框架上的 `framenavigated` 事件）。这是正确的——导航后，所有定位器都过时了。代理必须再次运行 `snapshot` 以获取新的引用。这是设计使然：过时的引用应该大声失败，而不是点击错误的元素。

### 引用过时检测

SPA 可以在不触发 `framenavigated` 的情况下突变 DOM（例如 React 路由器转换、标签页切换、模态框打开）。这使得即使页面 URL 没有更改，引用也会过时。为了捕获这一点，`resolveRef()` 在使用任何引用之前执行异步 `count()` 检查：

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → if count === 0: throw "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → if count > 0: return { locator }
```

这快速失败（约 5ms 开销），而不是让 Playwright 的 30 秒操作超时在缺失元素上过期。`RefEntry` 存储 `role` 和 `name` 元数据以及 Locator，以便错误消息可以告诉代理元素是什么。

### 光标交互引用 (@c)

`-C` 标志查找可点击但不在 ARIA 树中的元素——使用 `cursor: pointer` 样式的元素、带有 `onclick` 属性的元素或自定义 `tabindex` 的元素。这些在单独的命名空间中获得 `@c1`、`@c2` 引用。这捕获了框架渲染为 `<div>` 但实际上是按钮的自定义组件。

## 日志架构

三个环形缓冲区（每个 50,000 个条目，O(1) 推送）：

```
浏览器事件 → CircularBuffer (内存中) → 异步刷新到 .gstack/*.log
```

控制台消息、网络请求和对话框事件各有自己的缓冲区。刷新每 1 秒发生一次——服务器仅追加自上次刷新以来的新条目。这意味着：

- HTTP 请求处理永远不会被磁盘 I/O 阻塞
- 日志在服务器崩溃后仍然存在（最多 1 秒的数据丢失）
- 内存是有界的（50K 条目 × 3 个缓冲区）
- 磁盘文件是仅追加的，可由外部工具读取

`console`、`network` 和 `dialog` 命令从内存缓冲区读取，而不是从磁盘读取。磁盘文件用于事后调试。

## SKILL.md 模板系统

### 问题

SKILL.md 文件告诉 Claude 如何使用浏览命令。如果文档列出了不存在的标志，或错过了添加的命令，代理会遇到错误。手工维护的文档总是与代码脱节。

### 解决方案

```
SKILL.md.tmpl          (人工编写的散文 + 占位符)
       ↓
gen-skill-docs.ts      (读取源代码元数据)
       ↓
SKILL.md               (已提交，自动生成的部分)
```

模板包含需要人工判断的工作流程、提示和示例。占位符在构建时从源代码填充：

| 占位符 | 来源 | 生成内容 |
|-------------|--------|-------------------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | 分类命令表 |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | 带有示例的标志参考 |
| `{{PREAMBLE}}` | `gen-skill-docs.ts` | 启动块：更新检查、会话跟踪、贡献者模式、AskUserQuestion 格式 |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | 二进制发现 + 设置说明 |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | 针对 PR 目标技能的动态基础分支检测（ship, review, qa, plan-ceo-review） |
| `{{QA_METHODOLOGY}}` | `gen-skill-docs.ts` | /qa 和 /qa-only 的共享 QA 方法块 |
| `{{DESIGN_METHODOLOGY}}` | `gen-skill-docs.ts` | /plan-design-review 和 /design-review 的共享设计审计方法 |
| `{{REVIEW_DASHBOARD}}` | `gen-skill-docs.ts` | /ship 预检的审查就绪仪表板 |
| `{{TEST_BOOTSTRAP}}` | `gen-skill-docs.ts` | /qa, /ship, /design-review 的测试框架检测、引导、CI/CD 设置 |

这在结构上是健全的——如果代码中存在命令，它会出现在文档中。如果不存在，它就不会出现。

### 前言

每个技能都以一个在技能自身逻辑之前运行的 `{{PREAMBLE}}` 块开始。它在单个 bash 命令中处理四件事：

1. **更新检查** — 调用 `gstack-update-check`，报告是否有可用的升级。
2. **会话跟踪** — 触摸 `~/.gstack/sessions/$PPID` 并计算活动会话（最近 2 小时内修改的文件）。当运行 3+ 会话时，所有技能进入"ELI16 模式"——每个问题都会使用户重新了解上下文，因为他们在处理多个窗口。
3. **贡献者模式** — 从配置中读取 `gstack_contributor`。当为 true 时，代理在 gstack 本身行为不当时向 `~/.gstack/contributor-logs/` 提交临时现场报告。
4. **AskUserQuestion 格式** — 通用格式：上下文、问题、`RECOMMENDATION: Choose X because ___`、带字母的选项。在所有技能中一致。

### 为什么是已提交的，而不是运行时生成的？

三个原因：

1. **Claude 在技能加载时读取 SKILL.md**。用户调用 `/browse` 时没有构建步骤。文件必须已经存在且正确。
2. **CI 可以验证新鲜度**。`gen:skill-docs --dry-run` + `git diff --exit-code` 在合并前捕获过时的文档。
3. **Git blame 有效**。你可以看到命令何时添加以及在哪个提交中。

### 模板测试层级

| 层级 | 内容 | 成本 | 速度 |
|------|------|------|-------|
| 1 — 静态验证 | 解析 SKILL.md 中的每个 `$B` 命令，根据注册表验证 | 免费 | <2s |
| 2 — 通过 `claude -p` 的端到端 | 生成真实的 Claude 会话，运行每个技能，检查错误 | ~$3.85 | ~20min |
| 3 — LLM 作为判断者 | Sonnet 对文档的清晰度/完整性/可操作性进行评分 | ~$0.15 | ~30s |

层级 1 在每次 `bun test` 时运行。层级 2+3 被 `EVALS=1` 限制。理念是：免费捕获 95% 的问题，仅对判断调用使用 LLM。

## 命令分发

命令按副作用分类：

- **READ**（text, html, links, console, cookies, ...）：无突变。安全重试。返回页面状态。
- **WRITE**（goto, click, fill, press, ...）：突变页面状态。不是幂等的。
- **META**（snapshot, screenshot, tabs, chain, ...）：服务器级操作，不完全适合读/写。

这不仅仅是组织。服务器使用它进行分发：

```typescript
if (READ_COMMANDS.has(cmd))  → handleReadCommand(cmd, args, bm)
if (WRITE_COMMANDS.has(cmd)) → handleWriteCommand(cmd, args, bm)
if (META_COMMANDS.has(cmd))  → handleMetaCommand(cmd, args, bm, shutdown)
```

`help` 命令返回所有三个集合，以便代理可以自行发现可用命令。

## 错误哲学

错误是为 AI 代理准备的，不是为人类。每条错误消息都必须可操作：

- "Element not found" → "Element not found or not interactable. Run `snapshot -i` to see available elements."
- "Selector matched multiple elements" → "Selector matched multiple elements. Use @refs from `snapshot` instead."
- Timeout → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

Playwright 的原生错误通过 `wrapError()` 重写，以剥离内部堆栈跟踪并添加指导。代理应该能够读取错误并知道下一步该做什么，而无需人工干预。

### 崩溃恢复

服务器不尝试自我修复。如果 Chromium 崩溃（`browser.on('disconnected')`），服务器立即退出。CLI 在下次命令时检测到死亡的服务器并自动重启。这比尝试重新连接到半死的浏览器进程更简单、更可靠。

## 端到端测试基础设施

### 会话运行器 (`test/helpers/session-runner.ts`)

端到端测试生成 `claude -p` 作为完全独立的子进程——不是通过 Agent SDK，它不能嵌套在 Claude 代码会话中。运行器：

1. 将提示写入临时文件（避免 shell 转义问题）
2. 生成 `sh -c 'cat prompt | claude -p --output-format stream-json --verbose'`
3. 从 stdout 流式传输 NDJSON 以获取实时进度
4. 与可配置的超时竞争
5. 将完整的 NDJSON 转录解析为结构化结果

`parseNDJSON()` 函数是纯函数——没有 I/O，没有副作用——使其可独立测试。

### 可观测性数据流

```
  skill-e2e.test.ts
        │
        │ 生成 runId，将 testName + runId 传递给每个调用
        │
  ┌─────┼──────────────────────────────┐
  │     │                              │
  │  runSkillTest()              evalCollector
  │  (session-runner.ts)         (eval-store.ts)
  │     │                              │
  │  per tool call:              per addTest():
  │  ┌──┼──────────┐              savePartial()
  │  │  │          │                   │
  │  ▼  ▼          ▼                   ▼
  │ [HB] [PL]    [NJ]          _partial-e2e.json
  │  │    │        │             (atomic overwrite)
  │  │    │        │
  │  ▼    ▼        ▼
  │ e2e-  prog-  {name}
  │ live  ress   .ndjson
  │ .json .log
  │
  │  on failure:
  │  {name}-failure.json
  │
  │  ALL files in ~/.gstack-dev/
  │  Run dir: e2e-runs/{runId}/
  │
  │         eval-watch.ts
  │              │
  │        ┌─────┴─────┐
  │     read HB     read partial
  │        └─────┬─────┘
  │              ▼
  │        render dashboard
  │        (stale >10min? warn)
```

**分割所有权**：session-runner 拥有心跳（当前测试状态），eval-store 拥有部分结果（已完成的测试状态）。观察器读取两者。两个组件都不知道对方——它们仅通过文件系统共享数据。

**非致命一切**：所有可观测性 I/O 都包装在 try/catch 中。写入失败永远不会导致测试失败。测试本身是真相的来源；可观测性是尽力而为的。

**机器可读诊断**：每个测试结果包括 `exit_reason`（success, timeout, error_max_turns, error_api, exit_code_N）、`timeout_at_turn` 和 `last_tool_call`。这启用了如下的 `jq` 查询：
```bash
jq '.tests[] | select(.exit_reason == "timeout") | .last_tool_call' ~/.gstack-dev/evals/_partial-e2e.json
```

### 评估持久性 (`test/helpers/eval-store.ts`)

`EvalCollector` 累积测试结果并以两种方式写入：

1. **增量**：`savePartial()` 在每个测试后写入 `_partial-e2e.json`（原子：写入 `.tmp`，`fs.renameSync`）。在终止后仍然存在。
2. **最终**：`finalize()` 写入带时间戳的评估文件（例如 `e2e-20260314-143022.json`）。部分文件永远不会被清理——它与最终文件一起存在以用于可观测性。

`eval:compare` 比较两个评估运行。`eval:summary` 汇总 `~/.gstack-dev/evals/` 中所有运行的统计数据。

### 测试层级

| 层级 | 内容 | 成本 | 速度 |
|------|------|------|-------|
| 1 — 静态验证 | 解析 `$B` 命令，根据注册表验证，可观测性单元测试 | 免费 | <5s |
| 2 — 通过 `claude -p` 的端到端 | 生成真实的 Claude 会话，运行每个技能，扫描错误 | ~$3.85 | ~20min |
| 3 — LLM 作为判断者 | Sonnet 对文档的清晰度/完整性/可操作性进行评分 | ~$0.15 | ~30s |

层级 1 在每次 `bun test` 时运行。层级 2+3 被 `EVALS=1` 限制。理念是：免费捕获 95% 的问题，仅对判断调用和集成测试使用 LLM。

## 有意不在此处的内容

- **无 WebSocket 流式传输**。HTTP 请求/响应更简单，可通过 curl 调试，并且足够快。流式传输会为边际收益增加复杂性。
- **无 MCP 协议**。MCP 为每个请求添加 JSON 模式开销，需要持久连接。纯 HTTP + 纯文本输出在令牌上更轻，更易于调试。
- **无多用户支持**。每个工作区一个服务器，一个用户。令牌认证是深度防御，不是多租户。
- **无 Windows/Linux cookie 解密**。macOS 钥匙串是唯一支持的凭证存储。Linux（GNOME Keyring/kwallet）和 Windows（DPAPI）在架构上是可能的，但未实现。
- **无 iframe 支持**。Playwright 可以处理 iframe，但引用系统尚未跨框架边界。这是最常请求的缺失功能。