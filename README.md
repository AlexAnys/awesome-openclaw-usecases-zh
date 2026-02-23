<div align="center">

<img width="1500" height="500" alt="Image" src="https://github.com/user-attachments/assets/4ae57dfb-4f18-4677-9136-43bf93017250" />

<br/>
<br/>

<strong>OpenClaw 真实用例合集 — 探索人们如何在日常生活中使用 OpenClaw
</strong>
<br />
<br />

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
![用例数量](https://img.shields.io/badge/用例-29-blue?style=flat-square)
![语言](https://img.shields.io/badge/语言-中文-red?style=flat-square)
</div>

# Awesome OpenClaw 用例合集（中文版）

> **来源声明**
> 本仓库基于 [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases) 翻译和扩展，面向中文用户。感谢原作者 [hesamsheikh](https://github.com/hesamsheikh) 和所有社区贡献者。

解决 OpenClaw 普及的瓶颈：不是技能不够，而是找到**它能改善你生活的方式**。这是 [OpenClaw](https://github.com/openclaw/openclaw) 的社区真实使用案例合集。

---

## 新手入门指南

### OpenClaw 是什么？

OpenClaw（前身为 ClawdBot、MoltBot）是一个开源的 AI 智能体平台。简单来说，它就像一个 **7×24 小时在线的 AI 助手**，可以：

- 自动执行你安排的任务（比如每天早上给你发新闻摘要）
- 连接各种工具和服务（邮箱、日历、Telegram、Discord 等）
- 记住你的偏好，越用越懂你
- 甚至可以帮你写代码、管理服务器

### 核心概念速览

| 概念 | 英文 | 通俗解释 |
|------|------|----------|
| 技能 | Skill | 给 OpenClaw 安装的"插件"，让它能做特定的事（比如读 Reddit、发邮件） |
| 记忆 | Memory | OpenClaw 记住你说过的话和偏好，存在 Markdown 文件里 |
| 定时任务 | Cron Job | 让 OpenClaw 按时间表自动执行任务（比如每天早上 8 点） |
| 子智能体 | Sub-agent | OpenClaw 可以派出"分身"并行处理多个任务 |
| 心跳 | Heartbeat | 定期自动运行的检查任务，让 OpenClaw 持续关注重要事项 |
| 提示词 | Prompt | 你给 OpenClaw 的指令，告诉它要做什么 |

### 如何开始使用？

1. **安装 OpenClaw**：访问 [OpenClaw 官方仓库](https://github.com/openclaw/openclaw) 按照说明安装
2. **选择连接方式**：通过 Telegram、Discord、iMessage 等与你的 OpenClaw 对话
3. **安装技能**：根据需要从 [ClawHub](https://clawhub.ai) 安装对应技能
4. **开始使用**：从下方用例中选一个感兴趣的，按说明配置即可

### 如何阅读本合集？

每个用例文件通常包含以下部分：

- **痛点（Pain Point）**：这个用例解决什么问题
- **功能说明（What It Does）**：具体能做什么
- **所需技能（Skills Needed）**：需要安装哪些技能/插件
- **配置方法（How to Set It Up）**：手把手教你设置
- **关键洞察（Key Insights）**：使用过程中的经验总结

> 提示：代码块中的英文 prompt（提示词）建议保留英文使用，效果更佳。中文说明会帮你理解每段 prompt 的作用。

---

> **安全警告：** 此处引用的 OpenClaw 技能和第三方依赖可能存在安全风险。许多用例链接到社区构建的技能、插件和外部仓库，这些**未经本仓库维护者审核**。请始终审查技能源代码，检查请求的权限，并避免硬编码 API 密钥或凭证。您需对自己的安全负全部责任。

---

## 社交媒体

| 名称 | 描述 |
|------|------|
| [每日 Reddit 摘要](usecases/daily-reddit-digest.md) | 根据你的偏好，生成你喜爱的 subreddit 精选摘要 |
| [每日 YouTube 摘要](usecases/daily-youtube-digest.md) | 获取你关注频道的每日新视频摘要，不错过任何内容 |
| [X 账号分析](usecases/x-account-analysis.md) | 获取你的 X（原 Twitter）账号的定性分析 |
| [多源科技新闻摘要](usecases/multi-source-tech-news-digest.md) | 自动聚合 109+ 来源的科技新闻，支持质量评分和多渠道分发 |

## 创意与构建

| 名称 | 描述 |
|------|------|
| [目标驱动的自主任务](usecases/overnight-mini-app-builder.md) | 倾倒你的目标，让智能体自主生成、安排并完成每日任务，包括一夜之间构建迷你应用 |
| [YouTube 内容流水线](usecases/youtube-content-pipeline.md) | 为 YouTube 频道自动化视频创意发掘、研究和追踪 |
| [多智能体内容工厂](usecases/content-factory.md) | 在 Discord 中运行多智能体内容流水线，研究、写作和缩略图智能体在专用频道中协同工作 |

## 基础设施与 DevOps

| 名称 | 描述 |
|------|------|
| [n8n 工作流编排](usecases/n8n-workflow-orchestration.md) | 通过 webhook 将 API 调用委托给 n8n 工作流，智能体从不接触凭证 |
| [自愈家庭服务器](usecases/self-healing-home-server.md) | 运行始终在线的基础设施智能体，具有 SSH 访问、自动化定时任务和自愈能力 |

## 生产力

| 名称 | 描述 |
|------|------|
| [自主项目管理](usecases/autonomous-project-management.md) | 使用 STATE.yaml 模式协调多智能体项目，子智能体并行工作 |
| [多渠道 AI 客户服务](usecases/multi-channel-customer-service.md) | 将 WhatsApp、Instagram、邮件和 Google 评价统一到 AI 驱动的收件箱 |
| [基于电话的个人助理](usecases/phone-based-personal-assistant.md) | 通过电话或短信从任何手机访问你的 AI 智能体 |
| [收件箱整理](usecases/inbox-declutter.md) | 自动总结新闻通讯并发送摘要邮件 |
| [个人 CRM](usecases/personal-crm.md) | 自动从邮件和日历中发现并追踪联系人，支持自然语言查询 |
| [健康与症状追踪器](usecases/health-symptom-tracker.md) | 追踪食物摄入和症状以识别诱因，带有定期签到提醒 |
| [多渠道个人助理](usecases/multi-channel-assistant.md) | 从单个 AI 助理路由任务到 Telegram、Slack、邮件和日历 |
| [项目状态管理](usecases/project-state-management.md) | 事件驱动的项目追踪，自动捕获上下文，取代静态看板 |
| [动态仪表板](usecases/dynamic-dashboard.md) | 实时仪表板，并行从 API、数据库和社交媒体获取数据 |
| [Todoist 任务管理器](usecases/todoist-task-manager.md) | 将推理和进度日志同步到 Todoist，最大化智能体透明度 |
| [家庭日历与家务助理](usecases/family-calendar-household-assistant.md) | 聚合所有家庭日历到早间简报，监控消息获取预约，管理家庭库存 |
| [多智能体专业团队](usecases/multi-agent-team.md) | 通过单个 Telegram 聊天，运行多个专业智能体作为协调团队 |
| [定制早间简报](usecases/custom-morning-brief.md) | 获取完全定制的每日简报——新闻、任务、内容草稿和 AI 推荐操作 |
| [第二大脑](usecases/second-brain.md) | 向机器人发送任何内容来记住它，在自定义 Next.js 仪表板中搜索所有记忆 |
| [活动嘉宾确认](usecases/event-guest-confirmation.md) | 逐一自动呼叫嘉宾确认出席、收集备注并编译摘要 |

## 研究与学习

| 名称 | 描述 |
|------|------|
| [AI 财报追踪器](usecases/earnings-tracker.md) | 追踪科技/AI 财报，自动化预览、警报和详细摘要 |
| [个人知识库 (RAG)](usecases/knowledge-base-rag.md) | 将 URL、推文和文章拖入聊天，构建可搜索的知识库 |
| [市场研究与产品工厂](usecases/market-research-product-factory.md) | 从 Reddit 和 X 挖掘真实痛点，让 OpenClaw 构建解决方案 MVP |
| [语义记忆搜索](usecases/semantic-memory-search.md) | 为 OpenClaw 记忆文件添加向量驱动的语义搜索 |

## 金融与交易

| 名称 | 描述 |
|------|------|
| [Polymarket 自动驾驶](usecases/polymarket-autopilot.md) | 在预测市场上进行自动化模拟交易，带有回测、策略分析和每日绩效报告 |

---

## 贡献

我们欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

- 添加新的中文用例
- 改进现有翻译
- 提交中文独创用例

> 请只提交你已经使用过并验证有效的用例（至少使用一天）。我们重视真正让生活变得更好的真实想法！
>
> **注意：** 我们不接受与加密货币相关的用例。

---

**原始英文仓库**：[awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)
