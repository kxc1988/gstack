# 技能深入分析

每个 gstack 技能的详细指南 — 理念、工作流程和示例。

| 技能 | 你的专家 | 他们做什么 |
|-------|----------------|--------------|
| [`/office-hours`](#office-hours) | **YC 办公时间** | 从这里开始。六个强制问题，在你编写代码之前重新构建你的产品框架。对你的框架提出异议，挑战前提，生成实现替代方案。设计文档输入到每个下游技能中。 |
| [`/plan-ceo-review`](#plan-ceo-review) | **CEO / 创始人** | 重新思考问题。在请求中找到隐藏的 10 星产品。四种模式：扩展、选择性扩展、保持范围、减少。 |
| [`/plan-eng-review`](#plan-eng-review) | **工程经理** | 锁定架构、数据流、图表、边缘情况和测试。将隐藏的假设公之于众。 |
| [`/plan-design-review`](#plan-design-review) | **高级设计师** | 交互式计划模式设计审查。对每个维度评分 0-10，解释 10 分是什么样子，修复计划。在计划模式下工作。 |
| [`/design-consultation`](#design-consultation) | **设计合作伙伴** | 从头开始构建完整的设计系统。了解景观，提出创意风险，生成现实的产品模型。设计是所有其他阶段的核心。 |
| [`/review`](#review) | **高级工程师** | 找到通过 CI 但在生产中爆炸的错误。自动修复明显的错误。标记完整性差距。 |
| [`/debug`](#debug) | **调试器** | 系统根因调试。铁律：没有调查就没有修复。跟踪数据流，测试假设，3 次失败修复后停止。 |
| [`/design-review`](#design-review) | **会编码的设计师** | 现场视觉审计 + 修复循环。80 项审计，然后修复它发现的问题。原子提交，前后截图。 |
| [`/qa`](#qa) | **QA 负责人** | 测试你的应用，发现错误，用原子提交修复它们，重新验证。为每个修复自动生成回归测试。 |
| [`/qa-only`](#qa-only) | **QA 报告员** | 与 /qa 相同的方法，但仅报告。当你想要纯错误报告而不进行代码更改时使用。 |
| [`/ship`](#ship) | **发布工程师** | 同步主分支，运行测试，审计覆盖率，推送，打开 PR。如果你没有测试框架，它会引导测试框架。一个命令。 |
| [`/document-release`](#document-release) | **技术作家** | 更新所有项目文档以匹配你刚刚发布的内容。自动捕获过时的 README。 |
| [`/retro`](#retro) | **工程经理** | 团队感知的每周回顾。每人细分，发布 streak，测试健康趋势，增长机会。 |
| [`/browse`](#browse) | **QA 工程师** | 给代理眼睛。真正的 Chromium 浏览器，真正的点击，真正的截图。每个命令约 100ms。 |
| [`/setup-browser-cookies`](#setup-browser-cookies) | **会话管理器** | 将 cookie 从你真实的浏览器（Chrome、Arc、Brave、Edge）导入到无头会话中。测试认证页面。 |

---

## `/office-hours`

这是每个项目应该开始的地方。

在你计划之前，在你审查之前，在你编写代码之前 — 与 YC 风格的合作伙伴坐下来，思考你实际在构建什么。不是你认为你在构建什么。而是你*实际*在构建什么。

### 重新构建框架

这是在一个真实项目中发生的事情。用户说："我想为我的日历构建一个每日简报应用。" 合理的请求。然后它询问了痛点 — 具体例子，而不是假设。他们描述了一个错过事情的助手，跨多个 Google 账户的日历项目，带有过时信息，AI 缺陷的准备文档，需要永远追踪的错误位置的事件。

它回来时说：*"我要对你的框架提出异议，因为我认为你已经超越了它。你说'用于多 Google 日历管理的每日简报应用'。但你实际描述的是一个个人参谋长 AI。"*

然后它提取了用户没有意识到他们在描述的五个功能：

1. **监视你的日历** 跨所有账户，检测过时信息、缺失位置、权限差距
2. **生成真正的准备工作** — 不是物流摘要，而是*准备董事会会议、播客、筹款活动的智力工作*
3. **管理你的 CRM** — 你要见谁，关系是什么，他们想要什么，历史是什么
4. **优先安排你的时间** — 标记何时需要提前开始准备，主动阻止时间，按重要性排序事件
5. **用金钱换取杠杆** — 积极寻找委派或自动化的方法

那个重新构建框架改变了整个项目。他们即将构建一个日历应用。现在他们正在构建一个价值十倍的东西 — 因为技能倾听了他们的痛苦，而不是他们的功能请求。

### 前提挑战

重新构建框架后，它提出了让你验证的前提。不是"这听起来好吗？" — 而是关于产品的实际可证伪的声明：

1. 日历是锚定数据源，但价值在于顶部的智能层
2. 助手不会被替换 — 他们会获得超能力
3. 最窄的楔形是一个真正有效的每日简报
4. CRM 集成是必须的，不是可有可无的

你同意、不同意或调整。你接受的每个前提都成为设计文档中的承载点。

### 实现替代方案

然后它生成 2-3 个具体的实现方法，带有诚实的工作量估计：

- **方法 A：每日简报优先** — 最窄的楔形，明天发布，M 工作量（人类：~3 周 / CC：~2 天）
- **方法 B：CRM 优先** — 首先构建关系图，L 工作量（人类：~6 周 / CC：~4 天）
- **方法 C：完整愿景** — 一次完成所有事情，XL 工作量（人类：~3 个月 / CC：~1.5 周）

推荐 A，因为你从实际使用中学习。CRM 数据在第二周自然出现。

### 两种模式

**启动模式** — 适合创始人及在企业内部创业的人。你会得到六个从 YC 合作伙伴如何评估产品中提炼出的强制问题：需求现实、现状、绝望的具体性、最窄的楔形、观察与惊喜，以及未来适配性。这些问题故意让人不舒服。如果你不能说出一个需要你的产品的具体人，那是在编写任何代码之前需要学习的最重要的事情。

**构建者模式** — 适合黑客马拉松、副业项目、开源、学习和娱乐。你会得到一个热情的合作者，帮助你找到你的想法最酷的版本。什么会让有人说"哇"？什么是最快的路径到你可以分享的东西？问题是生成性的，不是询问性的。

### 设计文档

两种模式都以写入 `~/.gstack/projects/` 的设计文档结束 — 该文档直接输入到 `/plan-ceo-review` 和 `/plan-eng-review`。完整的生命周期现在是：`office-hours → plan → implement → review → QA → ship → retro`。

设计文档获得批准后，`/office-hours` 会反思它对你思考方式的注意 — 不是通用的赞美，而是对你在会话期间所说内容的具体回调。这些观察也出现在设计文档中，所以当你稍后重新阅读时，你会重新遇到它们。

---

## `/plan-ceo-review`

这是我的 **创始人模式**。

这是我希望模型以品味、野心、用户同理心和长期视野思考的地方。我不希望它按字面意思接受请求。我希望它首先问一个更重要的问题：

**这个产品实际是为了什么？**

我认为这是 **Brian Chesky 模式**。

重点不是实现明显的任务。重点是从用户的角度重新思考问题，找到感觉不可避免、令人愉快，甚至有点神奇的版本。

### 示例

假设我正在构建一个 Craigslist 风格的列表应用，我说：

> "让卖家为他们的物品上传照片。"

一个弱助手会添加一个文件选择器并保存图像。

那不是真正的产品。

在 `/plan-ceo-review` 中，我希望模型询问"照片上传"是否甚至是功能。也许真正的功能是帮助某人创建一个实际销售的列表。

如果那是真正的工作，整个计划都会改变。

现在模型应该问：

* 我们可以从照片中识别产品吗？
* 我们可以推断 SKU 或型号吗？
* 我们可以搜索网络并自动起草标题和描述吗？
* 我们可以提取规格、类别和价格比较吗？
* 我们可以建议哪张照片作为主图像最能转化吗？
* 我们可以检测上传的照片何时丑陋、黑暗、混乱或低信任度吗？
* 我们可以让体验感觉高级而不是像 2007 年的死表单吗？

这就是 `/plan-ceo-review` 为我做的。

它不只是问，"我如何添加这个功能？"
它问，**"这个请求中隐藏的 10 星产品是什么？"**

### 四种模式

- **范围扩展** — 大胆梦想。代理提出雄心勃勃的版本。每个扩展都作为你选择加入的个别决定呈现。热情推荐。
- **选择性扩展** — 将你当前的范围保持为基线，但看看还有什么可能。代理逐一展示机会，中性推荐 — 你挑选值得做的。
- **保持范围** — 对现有计划的最大严谨。不展示扩展。
- **范围减少** — 找到最小可行版本。削减其他一切。

愿景和决策被持久化到 `~/.gstack/projects/`，以便它们在对话之外生存。卓越的愿景可以升级到你仓库中的 `docs/designs/` 供团队使用。

---

## `/plan-eng-review`

这是我的 **工程经理模式**。

一旦产品方向正确，我想要完全不同的智能。我不想要更多的扩展思考。我不想要更多的"如果...会很酷"。我想要模型成为我最好的技术主管。

这个模式应该钉住：

* 架构
* 系统边界
* 数据流
* 状态转换
* 故障模式
* 边缘情况
* 信任边界
* 测试覆盖

对我来说一个令人惊讶的大解锁：**图表**。

当你强迫它们绘制系统时，LLM 变得更加完整。序列图、状态图、组件图、数据流图，甚至测试矩阵。图表迫使隐藏的假设公之于众。它们使挥手式规划更加困难。

所以 `/plan-eng-review` 是我希望模型构建能够承载产品愿景的技术脊柱的地方。

### 示例

以同一个列表应用示例为例。

假设 `/plan-ceo-review` 已经完成了它的工作。我们决定真正的功能不只是照片上传。它是一个智能列表流程，包括：

* 上传照片
* 识别产品
* 从网络丰富列表
* 起草强大的标题和描述
* 建议最佳主图像

现在 `/plan-eng-review` 接管。

现在我希望模型回答这样的问题：

* 上传、分类、丰富和草稿生成的架构是什么？
* 哪些步骤同步发生，哪些进入后台作业？
* 应用服务器、对象存储、视觉模型、搜索/丰富 API 和列表数据库之间的边界在哪里？
* 如果上传成功但丰富失败会发生什么？
* 如果产品识别置信度低会发生什么？
* 重试如何工作？
* 我们如何防止重复作业？
* 什么时候持久化什么，什么可以安全地重新计算？

这是我想要图表的地方 — 架构图、状态模型、数据流图、测试矩阵。图表迫使隐藏的假设公之于众。它们使挥手式规划更加困难。

那就是 `/plan-eng-review`。

不是"让想法更小"。
**让想法可构建。**

### 审查就绪仪表板

每次审查（CEO、Eng、Design）都会记录其结果。每次审查结束时，你会看到一个仪表板：

```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | YES      |
| CEO Review      |  1   | 2026-03-16 14:30    | CLEAR     | no       |
| Design Review   |  0   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: CLEARED — Eng Review passed                                |
+====================================================================+
```

Eng Review 是唯一必需的门（使用 `gstack-config set skip_eng_review true` 禁用）。CEO 和 Design 是信息性的 — 分别推荐用于产品和 UI 更改。

### 计划到 QA 流程

当 `/plan-eng-review` 完成测试审查部分时，它会将测试计划工件写入 `~/.gstack/projects/`。当你稍后运行 `/qa` 时，它会自动拾取该测试计划 — 你的工程审查直接进入 QA 测试，无需手动复制粘贴。

---

## `/plan-design-review`

这是我的 **高级设计师审查你的计划** — 在你编写一行代码之前。

大多数计划描述后端做什么，但从未指定用户实际看到什么。空状态？错误状态？加载状态？移动布局？AI 缺陷风险？这些决定被推迟到"在实现期间弄清楚" — 然后工程师发布"No items found."作为空状态，因为没有人指定更好的东西。

`/plan-design-review` 在规划期间捕获所有这些，此时修复成本低廉。

它的工作方式与 `/plan-ceo-review` 和 `/plan-eng-review` 一样 — 交互式，一次一个问题，带有 **STOP + AskUserQuestion** 模式。它对每个设计维度评分 0-10，解释 10 分是什么样子，然后编辑计划以达到目标。评分驱动工作：评分低 = 大量修复，评分高 = 快速通过。

对计划的七次通过：信息架构、交互状态覆盖、用户旅程、AI 缺陷风险、设计系统对齐、响应式/可访问性，以及未解决的设计决策。对于每次通过，它发现差距并直接修复它们（明显的）或让你做出设计选择（真正的权衡）。

### 示例

```
You:   /plan-design-review

Claude: Initial Design Rating: 4/10

        "This plan describes a user dashboard but never specifies
         what the user sees first. It says 'cards with icons' —
         which looks like every SaaS template. It mentions zero
         loading states, zero empty states, and no mobile behavior."

        Pass 1 (Info Architecture): 3/10
        "A 10 would define primary/secondary/tertiary content
         hierarchy for every screen."
        → Added information hierarchy section to plan

        Pass 2 (Interaction States): 2/10
        "The plan has 4 UI features but specifies 0 out of 20
         interaction states (4 features × 5 states each)."
        → Added interaction state table to plan

        Pass 4 (AI Slop): 4/10
        "The plan says 'clean, modern UI with cards and icons'
         and 'hero section with gradient'. These are the top 2
         AI-generated-looking patterns."
        → Rewrote UI descriptions with specific, intentional alternatives

        Overall: 4/10 → 8/10 after fixes
        "Plan is design-complete. Run /design-review after
         implementation for visual QA."
```

当你重新运行它时，已经达到 8+ 的部分会快速通过。低于 8 的部分会得到全面处理。对于实现后的现场视觉审计，请使用 `/design-review`。

---

## `/design-consultation`

这是我的 **设计合作伙伴模式**。

`/plan-design-review` 审计已经存在的站点。`/design-consultation` 是当你还没有任何东西时 — 没有设计系统，没有字体选择，没有调色板。你从零开始，想要一个高级设计师与你一起构建整个视觉标识。

这是一个对话，不是一个表单。代理询问你的产品、你的用户和你的受众。它思考你的产品需要传达什么 — 信任、速度、工艺、温暖，任何适合的东西 — 并从那里倒退到具体的选择。然后它提出一个完整、连贯的设计系统：美学方向、排版（3+ 字体，具有特定角色）、带有十六进制值的调色板、间距比例、布局方法和运动策略。每个建议都带有理由。每个选择都强化其他选择。

但连贯性是基本要求。每个开发工具仪表板看起来都一样 — 干净的无衬线字体，柔和的灰色，蓝色强调。它们都连贯。它们都容易被遗忘。一个看起来"不错"的产品和人们实际识别的产品之间的区别是 **刻意的创意风险**：标题的意外衬线，你的类别中没有人使用的大胆强调，使你的数据感觉权威而不是通风的更紧密间距。

这就是 `/design-consultation` 的真正意义。它不只是提出一个安全的系统。它提出安全选择和风险 — 并告诉你哪个是哪个。"这里是让你在你的类别中保持识字的选择。这里是我认为你应该打破常规的地方，以及为什么。" 你选择承担哪些风险。代理检查整个系统无论哪种方式仍然连贯。

如果你想要，代理会研究你的领域中已经存在的东西 — 拍摄真实站点的截图，分析它们的字体、颜色和间距 — 这样你可以在做出选择之前看到景观。这不是关于复制。这是关于进入球场，这样你知道惯例是什么，然后决定哪些值得打破。

在你同意系统后，它会生成一个交互式 HTML 预览页面 — 不只是色板和字体样本，而是现实的产品页面。如果你正在构建仪表板，你会看到一个带有侧边栏、数据表和统计卡片的仪表板。如果你正在构建营销站点，你会看到一个带有真实副本和 CTA 的英雄部分。所有内容都以你的设计系统呈现，带有你的产品名称，在浅色和深色模式下。在编写一行生产代码之前，你会看到你的产品可能的感觉。

然后它将 `DESIGN.md` 写入你的仓库根目录 — 你的项目的设计事实来源 — 并更新 `CLAUDE.md`，以便每个未来的 Claude 代码会话都尊重该系统。从那时起，`/design-review` 可以根据它进行审计，任何在你的前端工作的代理都知道规则。

### 示例

```
You:   /design-consultation

Claude: From the README, this looks like a developer analytics dashboard
        for engineering teams. Sound right? Tell me about the feeling
        you want — and would you like me to look at what's out there
        first so we can see the landscape?

You:    Yes, it's for eng managers. Show me what's out there.

Claude: [browses Linear, Vercel, Datadog, Posthog — takes screenshots]

        Here's the landscape. They all converge on clean sans-serifs
        and muted blue-gray palettes. Linear stands out with dark-first
        and tight spacing. The rest feel interchangeable.

        Here's my proposal — and where I'd play it safe vs. take risks:

        AESTHETIC: Industrial/Utilitarian — data-first, every pixel earns its place
        TYPOGRAPHY: Geist (body + UI), Instrument Serif (display), JetBrains Mono (data)
        COLOR: Restrained — slate neutrals, single teal accent (#0D9488)
        SPACING: 4px base, compact density
        LAYOUT: Grid-disciplined, 12-column
        MOTION: Minimal-functional — only transitions that aid comprehension

        SAFE CHOICES (match category expectations):
          Geist for body — your users already know this font from Vercel
          Slate neutrals — warm grays feel right for data-heavy UIs
          Grid-disciplined layout — dashboards need predictable alignment

        RISKS (where you'd stand out):
          Instrument Serif for display headings — nobody in dev tools
            uses a serif. It creates instant hierarchy contrast and
            signals "we care about craft" in a space full of sans-serifs.
          Single teal accent instead of blue — everyone else in this space uses blue.
            Teal is close enough to feel trustworthy, different enough
            to be recognizable.
          4px compact density — tighter than Linear, says "we respect
            your screen real estate."

        The safe choices keep you literate. The risks are where your
        product gets its own face. Which risks appeal to you?

You:    Love the serif risk. Let's see it.

Claude: Preview opened in your browser.
        [/tmp/design-consultation-preview-1710000000.html]

        The page shows a realistic dashboard: sidebar nav, header with
        avatar, stat cards, data table — all in your design system.
        Instrument Serif at hero scale, Geist in body text, JetBrains
        Mono in the data table. Teal accent on buttons and active states.
        Light and dark mode toggle in the corner.

        How does it feel? Want to adjust anything before I write DESIGN.md?

You:    Ship it.

Claude: Wrote DESIGN.md (typography, color, spacing, layout, motion).
        Updated CLAUDE.md with design system guidelines.
        All future sessions will respect these choices.
```

---

## `/design-review`

这是我的 **会编码的设计师模式**。

`/plan-design-review` 在实现前审查你的计划。`/design-review` 在实现后审计和修复现场站点。

它在你的现场站点上运行 80 项视觉审计，然后进入修复循环：对于每个设计发现，它定位源文件，进行最小的 CSS/样式更改，用 `style(design): FINDING-NNN` 提交，重新导航以验证，并拍摄前后截图。每个修复一个提交，完全可二分。

自我调节启发式针对设计工作进行了调整 — 仅 CSS 更改获得免费通行证（它们本质上是安全和可逆的），但对组件 JSX/TSX 文件的更改会占用风险预算。硬上限为 30 个修复。如果风险分数超过 20%，它会停止并询问。

### 示例

```
You:   /design-review https://myapp.com

Claude: [Runs full 80-item visual audit on the live site]
        Design Score: C  |  AI Slop Score: D
        12 findings (4 high, 5 medium, 3 polish)

        Fixing 9 design issues...

        style(design): FINDING-001 — replace 3-column icon grid with asymmetric layout
        style(design): FINDING-002 — add heading scale 48/32/24/18/16
        style(design): FINDING-003 — remove gradient hero, use bold typography
        style(design): FINDING-004 — add second font for headings
        style(design): FINDING-005 — vary border-radius by element role
        style(design): FINDING-006 — left-align body text, reserve center for headings
        style(design): FINDING-007 — add hover/focus states to all interactive elements
        style(design): FINDING-008 — add prefers-reduced-motion media query
        style(design): FINDING-009 — set max content width to 680px for body text

        Final audit:
        Design Score: C → B+  |  AI Slop Score: D → A
        9 fixes applied (8 verified, 1 best-effort). 3 deferred.

        [Report with before/after screenshots saved to .gstack/design-reports/]
```

九个提交，每个涉及一个问题。AI Slop 分数从 D 变为 A，因为三个最可识别的模式（渐变英雄、3 列网格、统一半径）都消失了。

---

## `/review`

这是我的 **偏执高级工程师模式**。

通过测试并不意味着分支是安全的。

`/review` 的存在是因为有一整类错误可以在 CI 中存活，仍然在生产中给你一拳。这个模式不是关于梦想更大。它不是关于让计划更漂亮。它是关于问：

**什么仍然可能打破？**

这是结构审计，不是风格挑剔。我希望模型寻找像这样的东西：

* N+1 查询
* 过时读取
* 竞争条件
* 糟糕的信任边界
* 缺失索引
* 转义错误
* 破坏的不变量
* 糟糕的重试逻辑
* 通过但错过真正失败模式的测试
* 被遗忘的枚举处理程序 — 添加新状态或类型常量，`/review` 会跟踪它通过代码库中的每个 switch 语句和允许列表，而不仅仅是你更改的文件

### 修复优先

发现得到行动，而不仅仅是列出。明显的机械修复（死代码、过时注释、N+1 查询）会自动应用 — 你会看到每个的 `[AUTO-FIXED] file:line Problem → what was done`。真正模糊的问题（安全、竞争条件、设计决策）会浮出水面供你调用。

### 完整性差距

`/review` 现在标记快捷实现，其中完整版本的成本不到 30 分钟的 CC 时间。如果你选择了 80% 的解决方案，而 100% 的解决方案是一个湖，不是海洋，审查会指出这一点。

### 示例

假设智能列表流程已实现，测试为绿色。

`/review` 仍然应该问：

* 我在渲染列表照片或草稿建议时是否引入了 N+1 查询？
* 我是信任客户端提供的文件元数据，还是验证实际文件？
* 两个标签页是否可以竞争并覆盖封面照片选择或项目详细信息？
* 失败的上传是否会在存储中永远留下孤立文件？
* "恰好一个主图像"规则在并发下是否会破裂？
* 如果丰富 API 部分失败，我是优雅降级还是保存垃圾？
* 我是否通过将网络数据拉入草稿生成而意外创建了提示注入或信任边界问题？

那就是 `/review` 的要点。

我不想要奉承。
我想要模型在生产事件发生之前想象它。

---

## `/debug`

当某件事坏了而你不知道为什么时，`/debug` 是你的系统调试器。它遵循铁律：**没有根因调查就没有修复。**

它不是猜测和修补，而是跟踪数据流，匹配已知的错误模式，并一次测试一个假设。如果三次修复尝试失败，它会停止并质疑架构，而不是 thrashing。这可以防止"让我再试一件事"的螺旋式浪费时间。

---

## `/qa`

这是我的 **QA 负责人模式**。

`/browse` 给代理眼睛。`/qa` 给它测试方法。

最常见的用例：你在功能分支上，刚完成编码，想要验证一切正常。只需说 `/qa` — 它读取你的 git diff，识别你的更改影响哪些页面和路由，启动浏览器，并测试每个页面。不需要 URL。不需要手动测试计划。

四种模式：

- **差异感知**（功能分支上自动）— 读取 `git diff main`，识别受影响的页面，专门测试它们
- **完整** — 系统探索整个应用。5-15 分钟。记录 5-10 个有充分证据的问题。
- **快速** (`--quick`) — 30 秒烟雾测试。主页 + 前 5 个导航目标。
- **回归** (`--regression baseline.json`) — 运行完整模式，然后与以前的基线进行比较。

### 自动回归测试

当 `/qa` 修复错误并验证它时，它会自动生成一个回归测试，捕获确切的中断场景。测试包括完整的归因追踪，回溯到 QA 报告。

### 示例

```
You:   /qa https://staging.myapp.com

Claude: [Explores 12 pages, fills 3 forms, tests 2 flows]

        QA Report: staging.myapp.com — Health Score: 72/100

        Top 3 Issues:
        1. CRITICAL: Checkout form submits with empty required fields
        2. HIGH: Mobile nav menu doesn't close after selecting an item
        3. MEDIUM: Dashboard chart overlaps sidebar below 1024px

        [Full report with screenshots saved to .gstack/qa-reports/]
```

**测试认证页面：** 首先使用 `/setup-browser-cookies` 导入你的真实浏览器会话，然后 `/qa` 可以测试登录后的页面。

---

## `/ship`

这是我的 **发布机器模式**。

一旦我决定要构建什么，确定技术计划，并进行认真的审查，我不想要更多的谈话。我想要执行。

`/ship` 是为最后一英里准备的。它是为准备好的分支准备的，不是为决定要构建什么准备的。

这是模型应该停止表现得像头脑风暴伙伴，开始表现得像纪律严明的发布工程师的地方：与主分支同步，运行正确的测试，确保分支状态正常，更新变更日志或版本控制（如果仓库需要），推送，并创建或更新 PR。

### 测试引导

如果你的项目没有测试框架，`/ship` 会设置一个 — 检测你的运行时，研究最佳框架，安装它，为你的实际代码编写 3-5 个真实测试，设置 CI/CD（GitHub Actions），并创建 TESTING.md。100% 测试覆盖率是目标 — 测试使氛围编码安全，而不是 yolo 编码。

### 覆盖率审计

每次 `/ship` 运行都会从你的差异构建代码路径图，搜索相应的测试，并生成带有质量星的 ASCII 覆盖率图。差距会自动生成测试。你的 PR 正文显示覆盖率：`Tests: 42 → 47 (+5 new)`。

### 审查门

`/ship` 在创建 PR 之前检查[审查就绪仪表板](#审查就绪仪表板)。如果 Eng Review 缺失，它会询问 — 但不会阻止你。决定按分支保存，所以你永远不会被重新询问。

当有趣的工作完成，只剩下无聊的发布工作时，很多分支会死亡。人类会拖延那部分。AI 不应该。

---

## `/document-release`

这是我的 **技术作家模式**。

在 `/ship` 创建 PR 但在它合并之前，`/document-release` 读取项目中的每个文档文件，并将其与差异交叉引用。它更新文件路径、命令列表、项目结构树，以及任何其他漂移的内容。有风险或主观的更改会作为问题浮出水面 — 其他一切都会自动处理。

```
You:   /document-release

Claude: Analyzing 21 files changed across 3 commits. Found 8 documentation files.

        README.md: updated skill count from 9 to 10, added new skill to table
        CLAUDE.md: added new directory to project structure
        CONTRIBUTING.md: current — no changes needed
        TODOS.md: marked 2 items complete, added 1 new item

        All docs updated and committed. PR body updated with doc diff.
```

它还会润色 CHANGELOG 语气（永远不会覆盖条目），清理已完成的 TODO，检查跨文档一致性，并且仅在适当时询问 VERSION 颠簸。

---

## `/retro`

这是我的 **工程经理模式**。

在周末结束时，我想知道实际发生了什么。不是氛围 — 数据。`/retro` 分析提交历史、工作模式和发布速度，并编写一个坦率的回顾。

它是团队感知的。它识别谁在运行命令，对你自己的工作给予最深的处理，然后分解每个贡献者，提供具体的赞美和成长机会。它计算指标，如提交、LOC、测试比率、PR 大小和修复比率。它从提交时间戳检测编码会话，找到热点文件，跟踪发布 streak，并识别本周最大的发布。

它还跟踪测试健康：测试文件总数、此期间添加的测试、回归测试提交，以及趋势增量。如果测试比率低于 20%，它会将其标记为增长区域。

### 示例

```
You:   /retro

Claude: Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm | Streak: 47d

        ## Your Week
        32 commits, +2.4k LOC, 41% tests. Peak hours: 9-11pm.
        Biggest ship: cookie import system (browser decryption + picker UI).
        What you did well: shipped a complete feature with encryption, UI, and
        18 unit tests in one focused push...

        ## Team Breakdown

        ### Alice
        12 commits focused on app/services/. Every PR under 200 LOC — disciplined.
        Opportunity: test ratio at 12% — worth investing before payment gets more complex.

        ### Bob
        3 commits — fixed the N+1 query on dashboard. Small but high-impact.
        Opportunity: only 1 active day this week — check if blocked on anything.

        [Top 3 team wins, 3 things to improve, 3 habits for next week]
```

它将 JSON 快照保存到 `.context/retros/`，以便下次运行可以显示趋势。

---

## `/browse`

这是我的 **QA 工程师模式**。

`/browse` 是闭合循环的技能。在它之前，代理可以思考和编码，但仍然半盲。它必须猜测 UI 状态、身份验证流程、重定向、控制台错误、空状态和损坏的布局。现在它可以去看看。

它是一个编译后的二进制文件，通过 [Microsoft 的 Playwright](https://playwright.dev/) 与持久 Chromium 守护进程通信。第一次调用启动浏览器（~3s）。之后的每次调用：~100-200ms。浏览器在命令之间保持运行，所以 cookie、标签页和 localStorage 会保留。

### 示例

```
You:   /browse staging.myapp.com — log in, test the signup flow, and check
       every page I changed in this branch

Claude: [18 tool calls, ~60 seconds]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "test@example.com"
        > browse fill @e3 "password123"
        > browse click @e5                    (Submit)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        Signup works. Redirected to onboarding. Now checking changed pages.

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        Dashboard loads. No console errors. Charts render with sample data.

        All 4 pages load correctly. No console errors. No broken layouts.
        Signup → onboarding → dashboard flow works end to end.
```

18 个工具调用，约一分钟。完整的 QA 通行证。没有打开浏览器。

**安全说明：** `/browse` 运行持久 Chromium 会话。Cookie、localStorage 和会话状态在命令之间保留。不要在敏感的生产环境中使用它，除非你打算这样做 — 它是一个具有真实状态的真实浏览器。会话在 30 分钟空闲时间后自动关闭。

有关完整的命令参考，请参阅 [BROWSER.md](../BROWSER.md)。

---

## `/setup-browser-cookies`

这是我的 **会话管理器模式**。

在 `/qa` 或 `/browse` 可以测试认证页面之前，它们需要 cookie。`/setup-browser-cookies` 不是每次都通过无头浏览器手动登录，而是直接从你的日常浏览器导入你的真实会话。

它自动检测已安装的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge），通过 macOS 钥匙串解密 cookie，并将它们加载到 Playwright 会话中。交互式选择器 UI 让你选择要导入的确切域 — 永远不会显示 cookie 值。

```
You:   /setup-browser-cookies

Claude: Cookie picker opened — select the domains you want to import
        in your browser, then tell me when you're done.

        [You pick github.com, myapp.com in the browser UI]

You:    done

Claude: Imported 2 domains (47 cookies). Session is ready.
```

或者完全跳过 UI：

```
You:   /setup-browser-cookies github.com

Claude: Imported 12 cookies for github.com from Comet.
```

---

## Greptile 集成

[Greptile](https://greptile.com) 是一家 YC 公司，可以自动审查你的 PR。它捕获真实的错误 — 竞争条件、安全问题、通过 CI 并在生产中爆炸的事情。它真的不止一次救了我的命。我爱这些家伙。

### 设置

在 [greptile.com](https://greptile.com) 上在你的 GitHub 仓库上安装 Greptile — 大约需要 30 秒。一旦它开始审查你的 PR，gstack 会自动拾取它的评论。无需额外配置。

### 工作原理

任何自动审查器的问题都是分类。Greptile 很好，但不是每个评论都是真实问题。有些是误报。有些标记你已经在三个提交前修复的东西。没有分类层，评论会堆积，你开始忽略它们 — 这违背了目的。

gstack 解决了这个问题。`/review` 和 `/ship` 现在是 Greptile 感知的。它们读取 Greptile 的评论，对每个进行分类，并采取行动：

- **有效问题** 添加到关键发现中，并在发布前修复
- **已经修复的问题** 得到自动回复，确认捕获
- **误报** 被推回 — 你确认，然后回复解释为什么它是错误的

结果是两层审查：Greptile 异步在 PR 上捕获东西，然后 `/review` 和 `/ship` 将这些发现分类为正常工作流的一部分。没有任何东西会被遗漏。

### 从历史中学习

你确认的每个误报都保存到 `~/.gstack/greptile-history.md`。未来的运行会自动跳过你的代码库的已知 FP 模式。`/retro` 随时间跟踪 Greptile 的击球率 — 所以你可以看到信号噪声比是否在改善。

### 示例

```
You:   /ship

Claude: [syncs main, runs tests, pre-landing review...]

        Greptile found 3 comments on this PR:

        [VALID] app/services/payment_service.rb:47 — Race condition:
        concurrent charges can double-debit. Recommend DB-level advisory
        lock around the charge block.

        [ALREADY FIXED] app/models/user.rb:12 — Missing null check
        on email before downcase.
        → Already fixed in commit abc1234. Auto-replying.

        [FALSE POSITIVE] lib/auth.rb:88 — "Token comparison should
        use constant-time comparison." We already use
        ActiveSupport::SecurityUtils.secure_compare here.

        Fixing the race condition in payment_service.rb...

        The auth token comment is a false positive.
        Options: A) Reply to Greptile explaining  B) Fix anyway  C) Ignore

You:    A

Claude: Replied to Greptile. All tests pass.
        PR: github.com/you/app/pull/42
```

三个 Greptile 评论。一个真正的修复。一个自动确认。一个误报被推回并回复。额外时间总计：约 30 秒。