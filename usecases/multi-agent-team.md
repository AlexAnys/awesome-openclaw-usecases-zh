# 多智能体专业团队（独立创始人配置）

独立创始人身兼数职——战略、开发、营销、销售、运营。在这些角色之间频繁切换会破坏深度工作。招聘既昂贵又缓慢。如果你能启动一个小型的专业 AI 智能体团队，每个智能体都有独特的角色和个性，全部通过单一聊天界面控制呢？

这个用例设置多个 OpenClaw 智能体作为一个协调团队，每个智能体专注于一个领域，通过共享记忆进行通信，并通过 Telegram 控制。

## 痛点

- **单一智能体无法胜任所有工作**：当同时处理战略、代码、营销研究和业务分析时，单个智能体的上下文窗口（context window）很快就会填满
- **缺乏专业化**：通用提示词产生通用输出——编码智能体不应该同时撰写营销文案
- **独立创始人的倦怠**：你需要的是一个团队，而不是又一个需要管理的工具。智能体应该在后台工作并呈现结果，而不是需要持续监管
- **知识孤岛**：营销研究的洞察不会自动影响开发优先级，除非你手动桥接它们

## 功能介绍

- **专业化智能体**：每个智能体都有独特的角色、个性和针对其领域优化的模型
- **共享记忆**：项目文档、目标和关键决策对所有智能体可访问——不会丢失任何信息
- **私有上下文**：每个智能体还维护自己的对话历史和领域特定笔记
- **统一控制面板**：所有智能体通过一个 Telegram 群组聊天访问——@标记你需要的智能体
- **每日定时任务**：智能体无需被要求即可主动工作——内容建议、竞品监控、指标跟踪
- **并行执行**：多个智能体可以同时处理独立任务

## 团队配置示例

### 智能体 1：Milo（战略负责人）

```text
## SOUL.md — Milo

You are Milo, the team lead. Confident, big-picture, charismatic.

Responsibilities:
- Strategic planning and prioritization
- Coordinating the other agents
- Weekly goal setting and OKR tracking
- Synthesizing insights from all agents into actionable decisions

Model: Claude Opus
Channel: Telegram (responds to @milo)

Daily tasks:
- 8:00 AM: Review overnight agent activity, post morning standup summary
- 6:00 PM: End-of-day recap with progress toward weekly goals
```

### 智能体 2：Josh（业务与增长）

```text
## SOUL.md — Josh

You are Josh, the business analyst. Pragmatic, straight to the point, numbers-driven.

Responsibilities:
- Pricing strategy and competitive analysis
- Growth metrics and KPI tracking
- Revenue modeling and unit economics
- Customer feedback analysis

Model: Claude Sonnet (fast, analytical)
Channel: Telegram (responds to @josh)

Daily tasks:
- 9:00 AM: Pull and summarize key metrics
- Track competitor pricing changes weekly
```

### 智能体 3：营销智能体

```text
## SOUL.md — Marketing Agent

You are the marketing researcher. Creative, curious, trend-aware.

Responsibilities:
- Content ideation and drafting
- Competitor social media monitoring
- Reddit/HN/X trend tracking for relevant topics
- SEO keyword research

Model: Gemini (strong at web research and long-context analysis)
Channel: Telegram (responds to @marketing)

Daily tasks:
- 10:00 AM: Surface 3 content ideas based on trending topics
- Monitor competitor Reddit/X mentions daily
- Weekly content calendar draft
```

### 智能体 4：开发智能体

```text
## SOUL.md — Dev Agent

You are the dev agent. Precise, thorough, security-conscious.

Responsibilities:
- Coding and architecture decisions
- Code review and quality checks
- Bug investigation and fixing
- Technical documentation

Model: Claude Opus / Codex (for implementation)
Channel: Telegram (responds to @dev)

Daily tasks:
- Check CI/CD pipeline health
- Review open PRs
- Flag technical debt items
```

## 所需技能

- `telegram` 技能用于共享控制界面
- `sessions_spawn` / `sessions_send` 用于多智能体协调
- 共享文件系统或笔记工具用于团队记忆
- 不同模型提供商的独立 API 密钥（如果使用混合模型）
- 一台 VPS 或始终在线的机器来运行智能体

## 设置方法

### 1. 共享记忆结构

```text
team/
├── GOALS.md           # 当前 OKR 和优先级（所有智能体可读）
├── DECISIONS.md       # 关键决策日志（仅追加）
├── PROJECT_STATUS.md  # 当前项目状态（所有智能体更新）
├── agents/
│   ├── milo/          # Milo 的私有上下文和笔记
│   ├── josh/          # Josh 的私有上下文
│   ├── marketing/     # 营销智能体的研究资料
│   └── dev/           # 开发智能体的技术笔记
```

### 2. Telegram 路由

配置一个所有智能体监听的 Telegram 群组，但每个智能体只在被 @标记时响应：

以下提示词配置 Telegram 群组中的智能体路由规则：

```text
## AGENTS.md — Telegram Routing

Telegram group: "Team"

Routing:
- @milo → Strategy agent (spawns/resumes milo session)
- @josh → Business agent (spawns/resumes josh session)
- @marketing → Marketing agent (spawns/resumes marketing session)
- @dev → Dev agent (spawns/resumes dev session)
- @all → Broadcast to all agents
- No tag → Milo (team lead) handles by default

Each agent:
1. Reads shared GOALS.md and PROJECT_STATUS.md for context
2. Reads its own private notes
3. Processes the message
4. Responds in Telegram
5. Updates shared files if the response involves a decision or status change
```

### 3. 定时任务

以下提示词设置团队的每日和每周自动化任务：

```text
## HEARTBEAT.md — Team Schedule

Daily:
- 8:00 AM: Milo posts morning standup (aggregates overnight agent activity)
- 9:00 AM: Josh pulls key metrics
- 10:00 AM: Marketing surfaces content ideas from trending topics
- 6:00 PM: Milo posts end-of-day recap

Ongoing:
- Dev: Monitor CI/CD health, review PRs as they come in
- Marketing: Reddit/X keyword monitoring (every 2 hours)
- Josh: Competitor pricing checks (weekly)

Weekly:
- Monday: Milo drafts weekly priorities (input from all agents)
- Friday: Josh compiles weekly metrics report
```

## 关键洞察

- **个性比你想象的更重要**：赋予智能体独特的名字和沟通风格，使得"与你的团队对话"变得自然，而不是与通用 AI 搏斗
- **共享记忆 + 私有上下文**：这种组合至关重要——智能体需要共同基础（目标、决策），但也需要自己的空间来积累领域专业知识
- **为正确的工作选择正确的模型**：不要用昂贵的推理模型来做关键词监控。将模型能力与任务复杂度匹配
- **定时任务是飞轮**：真正的价值在于智能体主动呈现洞察，而不是等你提问
- **从 2 个开始，而不是 4 个**：先配置一个负责人加一个专家，然后在发现瓶颈时再添加智能体

## 灵感来源

这个模式由 [Trebuh 在 X 上](https://x.com/iamtrebuh/status/2011260468975771862)描述，他是一位独立创始人，设置了 4 个 OpenClaw 智能体——Milo（战略负责人）、Josh（业务）、营销智能体和开发智能体——全部通过 VPS 上的单一 Telegram 聊天控制。每个智能体都有自己的个性、模型和定时任务，同时共享项目记忆。他将其描述为"一个 7x24 小时可用的真正小团队"。

这个模式在 [OpenClaw Showcase](https://openclaw.ai/showcase) 上也得到了验证，`@jdrhyne` 报告运行"15+ 个智能体，3 台机器，1 个 Discord 服务器——IT 基本上都是通过聊天构建的"，`@nateliason` 描述了一个多模型流水线（原型 -> 总结 -> 优化 -> 实现 -> 重复），在每个阶段使用不同的模型。另一位用户 `@danpeguine` 在同一个 WhatsApp 群组中运行两个不同的 OpenClaw 实例进行协作。

## 相关链接

- [OpenClaw 子智能体文档](https://github.com/openclaw/openclaw)
- [OpenClaw Telegram 技能](https://github.com/openclaw/openclaw)
- [OpenClaw Showcase](https://openclaw.ai/showcase)
- [Anthropic：构建有效的智能体](https://www.anthropic.com/research/building-effective-agents)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/multi-agent-team.md)
