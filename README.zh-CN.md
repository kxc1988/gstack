# gstack

大家好，我是 [加里·谭](https://x.com/garrytan)。我是 [Y Combinator](https://www.ycombinator.com/) 的总裁兼 CEO，在那里我与数千家初创公司合作过，包括 Coinbase、Instacart 和 Rippling，当时这些公司的创始人还只是在车库里的一两个人——现在这些公司的价值已经达到了数百亿美元。在 YC 之前，我设计了 Palantir 的标志，并且是那里最早的工程经理/产品经理/设计师之一。我共同创立了 Posterous，一个我们卖给 Twitter 的博客平台。2013 年，我在 YC 内部建立了社交网络 Bookface。长期以来，我一直以设计师、产品经理和工程经理的身份构建产品。

而现在，我正处于一个全新的时代。

在过去的 60 天里，我编写了**超过 60 万行生产代码**——其中 35% 是测试代码——并且我每天作为 CEO 职责的一部分，以兼职方式**每天完成 1 万到 2 万行可用代码**。这不是打字错误。我最近的 `/retro`（过去 7 天的开发者统计）在 3 个项目中：**添加了 140,751 行代码，362 次提交，约 115k 净代码行数**。模型每周都在显著改进。我们正处于一个真实的新时代的开端——一个人可以以过去需要 20 人团队的规模进行开发。

**2026 年 — 1,237 次贡献及计数：**

![GitHub 2026 年贡献 — 1,237 次贡献，1-3 月大幅加速](docs/images/github-2026.png)

**2013 年 — 我在 YC 构建 Bookface 时（772 次贡献）：**

![GitHub 2013 年贡献 — 在 YC 构建 Bookface 的 772 次贡献](docs/images/github-2013.png)

同一个人。不同的时代。差异在于工具。

**gstack 就是我的工具。** 它是我的开源软件工厂。它将 Claude 代码转变为一个你可以实际管理的虚拟工程团队——一个重新思考产品的 CEO，一个锁定架构的工程经理，一个捕捉 AI 缺陷的设计师，一个发现生产错误的偏执审查员，一个打开真实浏览器并点击你的应用的 QA 负责人，以及一个发布 PR 的发布工程师。十五个专家，都是斜杠命令，都是 Markdown，**全部免费，MIT 许可证，现在可用**。

我正在学习如何在 2026 年 3 月达到代理系统的极限，这是我的现场实验。我分享它是因为我希望全世界都和我一起踏上这段旅程。

分叉它。改进它。让它成为你的。不要仇视，要欣赏。

**这是为谁准备的：**
- **创始人与 CEO** — 尤其是仍然想要发布产品的技术型创始人。这是你如何像 20 人团队一样构建产品的方法。
- **首次使用 Claude 代码的用户** — gstack 是最好的开始方式。结构化角色而非空白提示。
- **技术主管和高级工程师** — 为每个 PR 带来严格的审查、QA 和发布自动化。

## 快速开始：你的前 10 分钟

1. 安装 gstack（30 秒 — 见下文）
2. 运行 `/office-hours` — 描述你正在构建的内容。在你编写代码之前，它会重新构建问题框架。
3. 对任何功能想法运行 `/plan-ceo-review`
4. 对任何有更改的分支运行 `/review`
5. 在你的 staging URL 上运行 `/qa`
6. 到此为止。你会知道这是否适合你。

预计在任何已设置测试的仓库中，首次有用运行时间不到 5 分钟。

**如果你只再读一个部分，请读这个。**

## 安装 — 只需 30 秒

**要求：** [Claude 代码](https://docs.anthropic.com/en/docs/claude-code)，[Git](https://git-scm.com/)，[Bun](https://bun.sh/) v1.0+

### 步骤 1：在你的机器上安装

打开 Claude 代码并粘贴以下内容。Claude 会完成其余的工作。

> 安装 gstack：运行 **`git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`** 然后在 CLAUDE.md 中添加一个 "gstack" 部分，说明使用 gstack 中的 /browse 技能进行所有网页浏览，永远不要使用 mcp__claude-in-chrome__* 工具，并列出可用的技能：/office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /retro, /debug, /document-release。然后询问用户是否还想将 gstack 添加到当前项目中，以便团队成员也能使用。

### 步骤 2：添加到你的仓库，以便团队成员也能使用（可选）

> 将 gstack 添加到这个项目：运行 **`cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup`** 然后在这个项目的 CLAUDE.md 中添加一个 "gstack" 部分，说明使用 gstack 中的 /browse 技能进行所有网页浏览，永远不要使用 mcp__claude-in-chrome__* 工具，列出可用的技能：/office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /retro, /debug, /document-release，并告诉 Claude 如果 gstack 技能不工作，运行 `cd .claude/skills/gstack && ./setup` 来构建二进制文件并注册技能。

真实文件会被提交到你的仓库（不是子模块），所以 `git clone` 可以正常工作。所有内容都存在于 `.claude/` 内部。没有任何内容会触及你的 PATH 或在后台运行。

## 观看它工作

```
你:    我想为我的日历构建一个每日简报应用。
你:    /office-hours
Claude: [询问关于痛点 — 具体例子，而非假设]

你:    多个 Google 日历，带有过时信息的事件，错误的位置。
        准备工作花费了很长时间，结果还不够好...

Claude: 我要对你的框架提出异议。你说"每日简报应用"。
        但你实际描述的是一个个人参谋长 AI。
        [提取你没有意识到你在描述的 5 个功能]
        [挑战 4 个前提 — 你同意、不同意或调整]
        [生成 3 个具有工作量估计的实现方法]
        建议：明天发布最窄的楔形，从实际使用中学习。
        完整愿景是一个 3 个月的项目 — 从真正有效的每日简报开始。
        [编写设计文档 → 自动输入下游技能]

你:    /plan-ceo-review
        [阅读设计文档，挑战范围，运行 10 部分审查]

你:    /plan-eng-review
        [数据流程、状态机、错误路径的 ASCII 图表]
        [测试矩阵、故障模式、安全问题]

你:    批准计划。退出计划模式。
        [在 11 个文件中编写 2,400 行代码。约 8 分钟。]

你:    /review
        [自动修复] 2 个问题。[询问] 竞争条件 → 你批准修复。

你:    /qa https://staging.myapp.com
        [打开真实浏览器，点击流程，发现并修复错误]

你:    /ship
        测试: 42 → 51 (+9 个新测试)。PR: github.com/you/app/pull/42
```

你说"每日简报应用"。代理说"你正在构建一个参谋长 AI"——因为它倾听了你的痛苦，而不是你的功能请求。然后它挑战了你的前提，生成了三种方法，推荐了最窄的楔形，并编写了一个设计文档，该文档被输入到每个下游技能中。八个命令。这不是一个副驾驶。这是一个团队。

## 冲刺

gstack 是一个过程，而不是工具的集合。技能的顺序是按照冲刺运行的方式排列的：

**思考 → 计划 → 构建 → 审查 → 测试 → 发布 → 反思**

每个技能都输入到下一个技能中。`/office-hours` 编写设计文档，`/plan-ceo-review` 读取该文档。`/plan-eng-review` 编写测试计划，`/qa` 接收该计划。`/review` 捕获 `/ship` 验证已修复的错误。没有任何内容会被遗漏，因为每个步骤都知道之前发生了什么。

一个冲刺，一个人，一个功能——使用 gstack 大约需要 30 分钟。但改变一切的是：你可以并行运行 10-15 个这样的冲刺。不同的功能，不同的分支，不同的代理——都在同一时间。这就是我如何在做实际工作的同时每天发布 10,000+ 行生产代码的方法。

| 技能 | 你的专家 | 他们做什么 |
|-------|----------------|--------------|
| `/office-hours` | **YC 办公时间** | 从这里开始。六个强制问题，在你编写代码之前重新构建你的产品框架。对你的框架提出异议，挑战前提，生成实现替代方案。设计文档输入到每个下游技能中。 |
| `/plan-ceo-review` | **CEO / 创始人** | 重新思考问题。在请求中找到隐藏的 10 星产品。四种模式：扩展、选择性扩展、保持范围、减少。 |
| `/plan-eng-review` | **工程经理** | 锁定架构、数据流、图表、边缘情况和测试。将隐藏的假设公之于众。 |
| `/plan-design-review` | **高级设计师** | 对每个设计维度评分 0-10，解释 10 分是什么样子，然后编辑计划以达到目标。AI 缺陷检测。互动式——每个设计选择一个 AskUserQuestion。 |
| `/design-consultation` | **设计合作伙伴** | 从头开始构建完整的设计系统。了解景观，提出创意风险，生成现实的产品模型。设计是所有其他阶段的核心。 |
| `/review` | **高级工程师** | 找到通过 CI 但在生产中爆炸的错误。自动修复明显的错误。标记完整性差距。 |
| `/debug` | **调试器** | 系统根因调试。铁律：没有调查就没有修复。跟踪数据流，测试假设，3 次失败修复后停止。 |
| `/design-review` | **会编码的设计师** | 与 /plan-design-review 相同的审计，然后修复它发现的问题。原子提交，前后截图。 |
| `/qa` | **QA 负责人** | 测试你的应用，发现错误，用原子提交修复它们，重新验证。为每个修复自动生成回归测试。 |
| `/qa-only` | **QA 报告员** | 与 /qa 相同的方法，但仅报告。当你想要纯错误报告而不进行代码更改时使用。 |
| `/ship` | **发布工程师** | 同步主分支，运行测试，审计覆盖率，推送，打开 PR。如果你没有测试框架，它会引导测试框架。一个命令。 |
| `/document-release` | **技术作家** | 更新所有项目文档以匹配你刚刚发布的内容。自动捕获过时的 README。 |
| `/retro` | **工程经理** | 团队感知的每周回顾。每人细分，发布 streak，测试健康趋势，增长机会。 |
| `/browse` | **QA 工程师** | 给代理眼睛。真正的 Chromium 浏览器，真正的点击，真正的截图。每个命令约 100ms。 |
| `/setup-browser-cookies` | **会话管理器** | 将 cookie 从你真实的浏览器（Chrome、Arc、Brave、Edge）导入到无头会话中。测试认证页面。 |

**[每个技能的深入分析，包括示例和理念 →](docs/skills.md)**

## 新功能及其重要性

**`/office-hours` 在你编写代码之前重新构建你的产品框架。** 你说"每日简报应用"。它倾听你的实际痛苦，对你的框架提出异议，告诉你你实际上正在构建一个个人参谋长 AI，挑战你的前提，并生成三个具有工作量估计的实现方法。它编写的设计文档直接输入到 `/plan-ceo-review` 和 `/plan-eng-review` 中——因此每个下游技能都从真正的清晰度开始，而不是模糊的功能请求。

**设计是核心。** `/design-consultation` 不仅仅是选择字体。它研究你的领域中存在的内容，提出安全选择和创意风险，生成你实际产品的逼真模型，并编写 `DESIGN.md`——然后 `/design-review` 和 `/plan-eng-review` 读取你选择的内容。设计决策贯穿整个系统。

**`/qa` 是一个巨大的解锁。** 它让我从 6 个并行工作者增加到 12 个。Claude 代码说 *"我看到了问题"*，然后实际修复它，生成回归测试，并验证修复——这改变了我的工作方式。代理现在有眼睛了。

**智能审查路由。** 就像在运行良好的初创公司一样：CEO 不必查看基础设施错误修复，后端更改不需要设计审查。gstack 跟踪运行了哪些审查，找出什么是合适的，并做明智的事情。审查就绪仪表板在你发布前告诉你你的状态。

**测试一切。** 如果你的项目没有测试框架，`/ship` 会从头开始引导测试框架。每次 `/ship` 运行都会产生覆盖率审计。每次 `/qa` 错误修复都会生成回归测试。100% 测试覆盖率是目标——测试使氛围编码安全，而不是 yolo 编码。

**`/document-release` 是你从未有过的工程师。** 它读取项目中的每个文档文件，交叉引用差异，并更新所有漂移的内容。README、ARCHITECTURE、CONTRIBUTING、CLAUDE.md、TODOS——所有内容都自动保持最新。

## 10-15 个并行冲刺

gstack 在一个冲刺中很强大。在同时运行十个冲刺时，它是变革性的。

[Conductor](https://conductor.build) 并行运行多个 Claude 代码会话——每个都在自己的隔离工作区中。一个会话在新想法上运行 `/office-hours`，另一个在 PR 上进行 `/review`，第三个实现功能，第四个在 staging 上运行 `/qa`，还有六个在其他分支上。所有这些都在同一时间。我经常运行 10-15 个并行冲刺——这是目前的实际最大值。

冲刺结构是使并行工作的原因。没有过程，十个代理就是十个混乱的来源。有了过程——思考、计划、构建、审查、测试、发布——每个代理都确切知道该做什么以及何时停止。你管理它们的方式就像 CEO 管理团队一样：检查重要的决策，让其余的运行。

---

## 来乘风破浪

这是 **免费的，MIT 许可，开源的，现在可用**。没有高级层。没有等待名单。没有附加条件。

我开源了我的开发方式，并且我正在这里积极升级我自己的软件工厂。你可以分叉它并使它成为你的。这就是全部意义。我希望每个人都加入这段旅程。

相同的工具，不同的结果——因为 gstack 为你提供结构化的角色和审查门，而不是通用的代理混乱。这种治理是快速发布和鲁莽发布之间的区别。

模型正在快速改进。现在弄清楚如何与它们一起工作的人——真正与它们一起工作，而不仅仅是涉猎——将拥有巨大的优势。这就是那个窗口。让我们开始吧。

十五个专家。所有斜杠命令。所有 Markdown。全部免费。**[github.com/garrytan/gstack](https://github.com/garrytan/gstack)** — MIT 许可证

> **我们正在招聘。** 想要每天发布 10K+ 代码行并帮助强化 gstack 吗？
> 来 YC 工作吧 — [ycombinator.com/software](https://ycombinator.com/software)
> 极具竞争力的薪资和股权。旧金山，Dogpatch 区。

## 文档

| 文档 | 涵盖内容 |
|-----|---------------|
| [技能深入分析](docs/skills.md) | 每个技能的理念、示例和工作流程（包括 Greptile 集成） |
| [架构](ARCHITECTURE.md) | 设计决策和系统内部结构 |
| [浏览器参考](BROWSER.md) | `/browse` 的完整命令参考 |
| [贡献](CONTRIBUTING.md) | 开发设置、测试、贡献者模式和开发模式 |
| [变更日志](CHANGELOG.md) | 每个版本的新内容 |

## 故障排除

**技能不显示？** `cd ~/.claude/skills/gstack && ./setup`

**`/browse` 失败？** `cd ~/.claude/skills/gstack && bun install && bun run build`

**安装过时？** 运行 `/gstack-upgrade` — 或在 `~/.gstack/config.yaml` 中设置 `auto_upgrade: true`

**Claude 说它看不到技能？** 确保你的项目的 `CLAUDE.md` 有一个 gstack 部分。添加这个：

```
## gstack
Use /browse from gstack for all web browsing. Never use mcp__claude-in-chrome__* tools.
Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /review, /ship, /browse, /qa, /qa-only, /design-review,
/setup-browser-cookies, /retro, /debug, /document-release.
```

## 许可证

MIT。永远免费。去构建一些东西。