# 浏览器 — 技术细节

本文档涵盖 gstack 无头浏览器的命令参考和内部工作原理。

## 命令参考

| 类别 | 命令 | 用途 |
|----------|----------|----------|
| 导航 | `goto`, `back`, `forward`, `reload`, `url` | 到达页面 |
| 读取 | `text`, `html`, `links`, `forms`, `accessibility` | 提取内容 |
| 快照 | `snapshot [-i] [-c] [-d N] [-s sel] [-D] [-a] [-o] [-C]` | 获取引用、差异、注释 |
| 交互 | `click`, `fill`, `select`, `hover`, `type`, `press`, `scroll`, `wait`, `viewport`, `upload` | 使用页面 |
| 检查 | `js`, `eval`, `css`, `attrs`, `is`, `console`, `network`, `dialog`, `cookies`, `storage`, `perf` | 调试和验证 |
| 视觉 | `screenshot [--viewport] [--clip x,y,w,h] [sel\|@ref] [path]`, `pdf`, `responsive` | 查看 Claude 看到的内容 |
| 比较 | `diff <url1> <url2>` | 发现环境之间的差异 |
| 对话框 | `dialog-accept [text]`, `dialog-dismiss` | 控制警报/确认/提示处理 |
| 标签页 | `tabs`, `tab`, `newtab`, `closetab` | 多页面工作流 |
| Cookies | `cookie-import`, `cookie-import-browser` | 从文件或真实浏览器导入 cookie |
| 多步骤 | `chain` (从 stdin 输入 JSON) | 一次调用中批量执行命令 |

所有选择器参数都接受 CSS 选择器、`snapshot` 后的 `@e` 引用或 `snapshot -C` 后的 `@c` 引用。总共 50+ 命令，加上 cookie 导入。

## 工作原理

gstack 的浏览器是一个编译后的 CLI 二进制文件，通过 HTTP 与本地持久 Chromium 守护进程通信。CLI 是一个瘦客户端 — 它读取状态文件，发送命令，并将响应打印到 stdout。服务器通过 [Playwright](https://playwright.dev/) 完成实际工作。

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude 代码                                                    │
│                                                                 │
│  "browse goto https://staging.myapp.com"                        │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    HTTP POST     ┌──────────────┐                 │
│  │ browse   │ ──────────────── │ Bun HTTP     │                 │
│  │ CLI      │  localhost:rand  │ server       │                 │
│  │          │  Bearer token    │              │                 │
│  │ compiled │ ◄──────────────  │  Playwright  │──── Chromium    │
│  │ binary   │  plain text      │  API calls   │    (headless)   │
│  └──────────┘                  └──────────────┘                 │
│   ~1ms startup                  persistent daemon               │
│                                 auto-starts on first call       │
│                                 auto-stops after 30 min idle    │
└─────────────────────────────────────────────────────────────────┘
```

### 生命周期

1. **首次调用**：CLI 检查项目根目录中的 `.gstack/browse.json` 是否有运行中的服务器。未找到 — 它在后台生成 `bun run browse/src/server.ts`。服务器通过 Playwright 启动无头 Chromium，选择随机端口（10000-60000），生成承载令牌，写入状态文件，并开始接受 HTTP 请求。这需要 ~3 秒。

2. **后续调用**：CLI 读取状态文件，发送带有承载令牌的 HTTP POST，打印响应。~100-200ms 往返。

3. **空闲关闭**：30 分钟无命令后，服务器关闭并清理状态文件。下次调用自动重启它。

4. **崩溃恢复**：如果 Chromium 崩溃，服务器立即退出（不自我修复 — 不要隐藏失败）。CLI 在下次命令时检测到死亡的服务器并启动一个新的。

### 关键组件

```
browse/
├── src/
│   ├── cli.ts              # 瘦客户端 — 读取状态文件，发送 HTTP，打印响应
│   ├── server.ts           # Bun.serve HTTP 服务器 — 将命令路由到 Playwright
│   ├── browser-manager.ts  # Chromium 生命周期 — 启动、标签页、引用映射、崩溃处理
│   ├── snapshot.ts         # 可访问性树 → @ref 分配 → Locator 映射 + 差异/注释/-C
│   ├── read-commands.ts    # 非突变命令（text, html, links, js, css, is, dialog 等）
│   ├── write-commands.ts   # 突变命令（click, fill, select, upload, dialog-accept 等）
│   ├── meta-commands.ts    # 服务器管理、链式命令、差异（通过 getCleanText 实现 DRY）、快照路由
│   ├── cookie-import-browser.ts  # 从真实 Chromium 浏览器解密并导入 cookie
│   ├── cookie-picker-routes.ts   # 交互式 cookie 选择器 UI 的 HTTP 路由
│   ├── cookie-picker-ui.ts       # 交互式 cookie 选择器的自包含 HTML/CSS/JS（深色主题，无框架）
│   └── buffers.ts          # CircularBuffer<T> + 控制台/网络/对话框捕获
├── test/                   # 集成测试 + HTML 夹具
└── dist/
    └── browse              # 编译后的二进制文件 (~58MB, Bun --compile)
```

### 快照系统

浏览器的关键创新是基于引用的元素选择，建立在 Playwright 的可访问性树 API 上：

1. `page.locator(scope).ariaSnapshot()` 返回类似 YAML 的可访问性树
2. 快照解析器为每个元素分配引用（`@e1`、`@e2`、...）
3. 对于每个引用，它构建一个 Playwright `Locator`（使用 `getByRole` + nth-child）
4. 引用到 Locator 的映射存储在 `BrowserManager` 上
5. 后来的命令如 `click @e3` 查找 Locator 并调用 `locator.click()`

没有 DOM 突变。没有注入的脚本。只有 Playwright 的原生可访问性 API。

**引用过时检测：** SPA 可以在不导航的情况下突变 DOM（React 路由器、标签页切换、模态框）。当这种情况发生时，从之前的 `snapshot` 收集的引用可能指向不再存在的元素。为了处理这一点，`resolveRef()` 在使用任何引用之前运行异步 `count()` 检查 — 如果元素计数为 0，它会立即抛出一条消息，告诉代理重新运行 `snapshot`。这快速失败（~5ms），而不是等待 Playwright 的 30 秒操作超时。

**扩展快照功能：**
- `--diff` (`-D`)：将每个快照存储为基线。在下次 `-D` 调用时，返回显示变化的统一差异。使用此功能验证操作（点击、填写等）是否实际有效。
- `--annotate` (`-a`)：在每个引用的边界框处注入临时覆盖 div，拍摄带有可见引用标签的屏幕截图，然后移除覆盖。使用 `-o <path>` 控制输出路径。
- `--cursor-interactive` (`-C`)：使用 `page.evaluate` 扫描非 ARIA 交互元素（带有 `cursor:pointer` 的 div、`onclick`、`tabindex>=0`）。使用确定性的 `nth-child` CSS 选择器分配 `@c1`、`@c2`... 引用。这些是 ARIA 树错过但用户仍然可以点击的元素。

### 截图模式

`screenshot` 命令支持四种模式：

| 模式 | 语法 | Playwright API |
|------|--------|----------------|
| 整页（默认） | `screenshot [path]` | `page.screenshot({ fullPage: true })` |
| 仅视口 | `screenshot --viewport [path]` | `page.screenshot({ fullPage: false })` |
| 元素裁剪 | `screenshot "#sel" [path]` 或 `screenshot @e3 [path]` | `locator.screenshot()` |
| 区域裁剪 | `screenshot --clip x,y,w,h [path]` | `page.screenshot({ clip })` |

元素裁剪接受 CSS 选择器（`.class`、`#id`、`[attr]`）或来自 `snapshot` 的 `@e`/`@c` 引用。自动检测：`@e`/`@c` 前缀 = 引用，`.`/`#`/`[` 前缀 = CSS 选择器，`--` 前缀 = 标志，其他 = 输出路径。

互斥：`--clip` + 选择器和 `--viewport` + `--clip` 都会抛出错误。未知标志（例如 `--bogus`）也会抛出错误。

### 认证

每个服务器会话生成一个随机 UUID 作为承载令牌。令牌被写入状态文件（`.gstack/browse.json`），权限为 chmod 600。每个 HTTP 请求必须包含 `Authorization: Bearer <token>`。这可以防止同一机器上的其他进程控制浏览器。

### 控制台、网络和对话框捕获

服务器挂钩到 Playwright 的 `page.on('console')`、`page.on('response')` 和 `page.on('dialog')` 事件。所有条目都保存在 O(1) 环形缓冲区（每个容量 50,000）中，并通过 `Bun.write()` 异步刷新到磁盘：

- 控制台：`.gstack/browse-console.log`
- 网络：`.gstack/browse-network.log`
- 对话框：`.gstack/browse-dialog.log`

`console`、`network` 和 `dialog` 命令从内存缓冲区读取，而不是从磁盘读取。

### 对话框处理

对话框（警报、确认、提示）默认自动接受，以防止浏览器锁定。`dialog-accept` 和 `dialog-dismiss` 命令控制此行为。对于提示，`dialog-accept <text>` 提供响应文本。所有对话框都记录到对话框缓冲区，包含类型、消息和采取的操作。

### JavaScript 执行 (`js` 和 `eval`)

`js` 运行单个表达式，`eval` 运行 JS 文件。两者都支持 `await` — 包含 `await` 的表达式会自动包装在异步上下文中：

```bash
$B js "await fetch('/api/data').then(r => r.json())"  # 有效
$B js "document.title"                                  # 也有效（不需要包装）
$B eval my-script.js                                    # 带有 await 的文件也有效
```

对于 `eval` 文件，单行文件直接返回表达式值。多行文件在使用 `await` 时需要显式 `return`。包含 "await" 的注释不会触发包装。

### 多工作区支持

每个工作区获得自己的隔离浏览器实例，带有自己的 Chromium 进程、标签页、cookie 和日志。状态存储在项目根目录内的 `.gstack/` 中（通过 `git rev-parse --show-toplevel` 检测）。

| 工作区 | 状态文件 | 端口 |
|-----------|------------|------|
| `/code/project-a` | `/code/project-a/.gstack/browse.json` | 随机（10000-60000） |
| `/code/project-b` | `/code/project-b/.gstack/browse.json` | 随机（10000-60000） |

无端口冲突。无共享状态。每个项目完全隔离。

### 环境变量

| 变量 | 默认值 | 描述 |
|----------|---------|-------------|
| `BROWSE_PORT` | 0（随机 10000-60000） | HTTP 服务器的固定端口（调试覆盖） |
| `BROWSE_IDLE_TIMEOUT` | 1800000（30 分钟） | 空闲关闭超时（毫秒） |
| `BROWSE_STATE_FILE` | `.gstack/browse.json` | 状态文件路径（CLI 传递给服务器） |
| `BROWSE_SERVER_SCRIPT` | 自动检测 | server.ts 路径 |

### 性能

| 工具 | 首次调用 | 后续调用 | 每次调用的上下文开销 |
|------|-----------|-----------------|--------------------------|
| Chrome MCP | ~5s | ~2-5s | ~2000 令牌（模式 + 协议） |
| Playwright MCP | ~3s | ~1-3s | ~1500 令牌（模式 + 协议） |
| **gstack browse** | **~3s** | **~100-200ms** | **0 令牌**（纯文本 stdout） |

上下文开销差异快速累积。在 20 命令的浏览器会话中，MCP 工具仅在协议框架上就消耗 30,000-40,000 令牌。gstack 消耗零。

### 为什么选择 CLI 而不是 MCP？

MCP（模型上下文协议）对于远程服务很有效，但对于本地浏览器自动化，它只添加纯粹的开销：

- **上下文膨胀**：每次 MCP 调用都包含完整的 JSON 模式和协议框架。一个简单的"获取页面文本"的成本是它应该的 10 倍以上的上下文令牌。
- **连接脆弱性**：持久 WebSocket/stdio 连接会断开并无法重新连接。
- **不必要的抽象**：Claude 代码已经有一个 Bash 工具。一个打印到 stdout 的 CLI 是最简单的可能接口。

### 开发

#### 先决条件

- [Bun](https://bun.sh/) v1.0+
- Playwright 的 Chromium（由 `bun install` 自动安装）

#### 快速开始

```bash
bun install              # 安装依赖 + Playwright Chromium
bun test                 # 运行集成测试 (~3s)
bun run dev <cmd>        # 从源代码运行 CLI（无编译）
bun run build            # 编译到 browse/dist/browse
```

#### 开发模式 vs 编译二进制文件

在开发期间，使用 `bun run dev` 而不是编译后的二进制文件。它直接用 Bun 运行 `browse/src/cli.ts`，所以你无需编译步骤即可获得即时反馈：

```bash
bun run dev goto https://example.com
bun run dev text
bun run dev snapshot -i
bun run dev click @e3
```

编译后的二进制文件（`bun run build`）仅用于分发。它使用 Bun 的 `--compile` 标志在 `browse/dist/browse` 生成一个单一的 ~58MB 可执行文件。

#### 运行测试

```bash
bun test                         # 运行所有测试
bun test browse/test/commands              # 仅运行命令集成测试
bun test browse/test/snapshot              # 仅运行快照测试
bun test browse/test/cookie-import-browser # 仅运行 cookie 导入单元测试
```

测试启动本地 HTTP 服务器（`browse/test/test-server.ts`），提供来自 `browse/test/fixtures/` 的 HTML 夹具，然后针对这些页面执行 CLI 命令。3 个文件共 203 个测试，总计约 15 秒。

#### 源映射

| 文件 | 角色 |
|------|------|
| `browse/src/cli.ts` | 入口点。读取 `.gstack/browse.json`，向服务器发送 HTTP，打印响应。 |
| `browse/src/server.ts` | Bun HTTP 服务器。将命令路由到正确的处理程序。管理空闲超时。 |
| `browse/src/browser-manager.ts` | Chromium 生命周期 — 启动、标签页管理、引用映射、崩溃检测。 |
| `browse/src/snapshot.ts` | 解析可访问性树，分配 `@e`/`@c` 引用，构建 Locator 映射。处理 `--diff`、`--annotate`、`-C`。 |
| `browse/src/read-commands.ts` | 非突变命令：`text`、`html`、`links`、`js`、`css`、`is`、`dialog`、`forms` 等。导出 `getCleanText()`。 |
| `browse/src/write-commands.ts` | 突变命令：`goto`、`click`、`fill`、`upload`、`dialog-accept`、`useragent`（带上下文重建）等。 |
| `browse/src/meta-commands.ts` | 服务器管理、链式命令路由、差异（通过 `getCleanText` 实现 DRY）、快照委托。 |
| `browse/src/cookie-import-browser.ts` | 通过 macOS 钥匙串 + PBKDF2/AES-128-CBC 解密 Chromium cookie。自动检测已安装的浏览器。 |
| `browse/src/cookie-picker-routes.ts` | `/cookie-picker/*` 的 HTTP 路由 — 浏览器列表、域搜索、导入、删除。 |
| `browse/src/cookie-picker-ui.ts` | 交互式 cookie 选择器的自包含 HTML 生成器（深色主题，无框架）。 |
| `browse/src/buffers.ts` | `CircularBuffer<T>`（O(1) 环形缓冲区）+ 控制台/网络/对话框捕获，带有异步磁盘刷新。 |

#### 部署到活动技能

活动技能位于 `~/.claude/skills/gstack/`。进行更改后：

1. 推送你的分支
2. 在技能目录中拉取：`cd ~/.claude/skills/gstack && git pull`
3. 重建：`cd ~/.claude/skills/gstack && bun run build`

或者直接复制二进制文件：`cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`

#### 添加新命令

1. 在 `read-commands.ts`（非突变）或 `write-commands.ts`（突变）中添加处理程序
2. 在 `server.ts` 中注册路由
3. 在 `browse/test/commands.test.ts` 中添加测试用例，如果需要，添加 HTML 夹具
4. 运行 `bun test` 验证
5. 运行 `bun run build` 编译