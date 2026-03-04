<div align="center">

<img width="1500" height="500" alt="OpenClaw AI 智能体最佳用例合集" src="https://github.com/user-attachments/assets/4ae57dfb-4f18-4677-9136-43bf93017250" />

<br/>
<br/>

<h3>OpenClaw AI 智能体最佳真实用例大全</h3>

<p>39 个经过验证的真实场景，手把手教你用 AI 智能体自动化工作与生活</p>

<br/>

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
![用例数量](https://img.shields.io/badge/用例-39-blue?style=flat-square)
![中文](https://img.shields.io/badge/语言-简体中文-red?style=flat-square)
![新手友好](https://img.shields.io/badge/难度-新手友好-green?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

**[快速开始](#-新手入门指南)** · **[用例目录](#-目录)** · **[贡献指南](CONTRIBUTING.md)**

</div>

---

# Awesome OpenClaw 最佳用例合集（中文版）

> 解决 OpenClaw 普及的瓶颈：不是 ~~技能~~，而是找到 **它能改善你生活的方式**。
>
> 一份面向中文用户的 OpenClaw 真实使用案例合集，包含社区验证的国际用例的中国适配，国内生态特色用例，从零入门。

<details>
<summary><strong>📖 来源声明</strong></summary>

本仓库基于 [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases) 翻译和扩展，面向中文用户。感谢原作者 [hesamsheikh](https://github.com/hesamsheikh) 和所有社区贡献者。

</details>

> **2026.3.4 更新**：中国特色用例扩充至 13 个——新增 A 股财报追踪、多渠道客服、创意验证器等国际用例的国内适配，按场景分类展示。详见 [中国特色用例](#-中国特色用例)。
>
> **2026.3.1 更新**：新增 9 个中国特色用例——飞书、钉钉、企业微信三大 IM 入口全覆盖，以及小红书自动化、办公套件等场景。

---

## 🗂 目录

- [新手入门指南](#-新手入门指南)
- [中国特色用例](#-中国特色用例) — 适配国内生态的精选用例（飞书、钉钉、企业微信、AKShare 等）
- [社交媒体](#社交媒体) — 自动获取 Reddit、YouTube、X 等平台的精选内容
- [创意与构建](#创意与构建) — 让 AI 帮你创作内容、构建应用
- [基础设施与 DevOps](#基础设施与-devops) — 服务器自愈、工作流自动化
- [生产力](#生产力) — 邮件整理、日程管理、客户服务、个人助理
- [研究与学习](#研究与学习) — 知识库、市场调研、财报追踪
- [金融与交易](#金融与交易) — 预测市场模拟交易

---

## 🚀 新手入门指南

### OpenClaw 是什么？

[OpenClaw](https://github.com/openclaw/openclaw)（前身为 ClawdBot、MoltBot）是一个开源的 **AI 智能体**平台。

和普通的 AI 聊天不同，OpenClaw 能够：

| 普通 AI 聊天 | OpenClaw 智能体 |
|:---:|:---:|
| 你问一句，它答一句 | **主动**帮你完成任务 |
| 关闭窗口就忘了一切 | **永久记住**你的偏好和指令 |
| 只能在网页里对话 | 连接邮箱、日历、Telegram、Discord 等**多个平台** |
| 需要你手动触发 | 按你设定的时间**自动运行** |
| 一次只能做一件事 | 派出多个**子智能体并行工作** |

简单来说，它是一个 **7×24 小时在线的 AI 员工**。

### 核心概念一看就懂

| 概念 | 英文 | 一句话解释 | 生活类比 |
|:---:|:---:|---|---|
| 技能 | Skill | 给 OpenClaw 安装的"插件" | 像给手机装 App |
| 记忆 | Memory | 记住你说过的话和偏好 | 像一个越来越懂你的老助理 |
| 定时任务 | Cron Job | 按时间表自动执行 | 像设了个闹钟 |
| 子智能体 | Sub-agent | 派出"分身"并行处理 | 像带了一个小团队 |
| 心跳 | Heartbeat | 定期自动检查状态 | 像保安巡逻 |
| 提示词 | Prompt | 你给它的指令 | 像给员工安排工作 |

### 三步开始使用

```
1️⃣  安装 OpenClaw → github.com/openclaw/openclaw
2️⃣  连接你的 Telegram / Discord / iMessage / 飞书 / 钉钉 / 企业微信
3️⃣  从下方选一个用例，复制提示词，粘贴给你的 OpenClaw
```

就这么简单。

### 如何阅读本合集？

每个用例都按统一格式组织：

| 章节 | 内容 |
|------|------|
| **痛点** | 这个用例解决什么问题 |
| **功能说明** | 具体能做什么 |
| **所需技能** | 需要安装哪些插件 |
| **配置方法** | 手把手教你设置（含可复制的提示词） |
| **关键洞察** | 踩过的坑和最佳实践 |

> 💡 **提示**：代码块中的英文 prompt（提示词）建议直接复制使用，效果最佳。每段 prompt 上方都有中文说明帮你理解其作用。

---

> ⚠️ **安全警告**：此处引用的 OpenClaw 技能和第三方依赖可能存在安全风险。许多用例链接到社区构建的技能、插件和外部仓库，这些**未经本仓库维护者审核**。请始终审查技能源代码，检查请求的权限，避免硬编码 API 密钥或凭证。您需对自己的安全负全部责任。

---

> 难度说明：⭐ 复制粘贴即可用 · ⭐⭐ 需要简单配置 · ⭐⭐⭐ 需要一定技术基础

---

## 🇨🇳 中国特色用例

> 专为中国工具生态设计或适配的用例，使用飞书、钉钉、企业微信、小红书等国内平台。标注"适配"的用例在国际版基础上增加了国内方案。

**IM 接入**

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [飞书 AI 助手](usecases/cn-feishu-ai-assistant.md) | 把 OpenClaw 部署为飞书机器人，在对话中直接触发 AI 任务，支持文档自动化 | ⭐⭐ |
| [钉钉 AI 助手](usecases/cn-dingtalk-ai-assistant.md) | 把 OpenClaw 部署为钉钉机器人，Stream 模式无需公网 IP | ⭐⭐ |
| [企业微信 AI 助手](usecases/cn-wecom-ai-assistant.md) | 在企业微信中使用 AI，通过微信插件让个人微信也能用 | ⭐⭐ |

**生产力与办公**

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [早间简报（适配）](usecases/custom-morning-brief.md) | 每日定时推送简报到飞书/钉钉，支持中文新闻源和 cron 配置 | ⭐ |
| [办公自动化套件](usecases/cn-office-automation.md) | 邮件管理、文件整理、会议纪要、周报生成，支持 163/QQ/Outlook | ⭐⭐ |
| [会议纪要与待办自动化（适配）](usecases/meeting-notes-action-items.md) | 会议转录自动生成纪要并创建任务（飞书妙记/腾讯会议/钉钉） | ⭐⭐ |
| [多渠道 AI 客户服务（适配）](usecases/multi-channel-customer-service.md) | 企业微信/抖音/小红书多渠道客服自动化 | ⭐⭐⭐ |

**内容创作**

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [小红书内容自动化](usecases/cn-xiaohongshu-automation.md) | 从选题、文案、封面图到定时发布的全流程自动化 | ⭐ |
| [播客制作流水线（适配）](usecases/podcast-production-pipeline.md) | 从选题到发布的全流程播客制作自动化（小宇宙/喜马拉雅/B站） | ⭐⭐ |

**研究与金融**

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [AI 财报追踪器（适配）](usecases/earnings-tracker.md) | A 股财报追踪，AKShare 免费数据源 + 业绩预告/快报自动化 | ⭐⭐ |
| [开发前创意验证器（适配）](usecases/pre-build-idea-validator.md) | 编码前自动扫描竞品（百度指数/微信指数/V2EX/少数派） | ⭐⭐ |

**高级场景**

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [多智能体协作操作系统](usecases/cn-multi-agent-operating-system.md) | 把 OpenClaw 变成专业分工、协同、稳定迭代的智能体系统 | ⭐⭐⭐ |

---

## 社交媒体

> 自动聚合你关心的内容，告别信息焦虑

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [每日 Reddit 摘要](usecases/daily-reddit-digest.md) | 根据你的偏好，生成你喜爱的 subreddit 精选摘要 | ⭐ |
| [每日 YouTube 摘要](usecases/daily-youtube-digest.md) | 获取你关注频道的每日新视频摘要，不错过任何内容 | ⭐ |
| [X 账号分析](usecases/x-account-analysis.md) | 获取你的 X（原 Twitter）账号的定性分析 | ⭐⭐ |
| [多源科技新闻摘要](usecases/multi-source-tech-news-digest.md) | 自动聚合 109+ 来源的科技新闻，支持质量评分和多渠道分发 | ⭐⭐ |

## 创意与构建

> 让 AI 智能体帮你创作内容、从零构建产品

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [目标驱动的自主任务](usecases/overnight-mini-app-builder.md) | 告诉 AI 你的目标，它自动拆解并每天执行，还能一夜之间造出迷你应用 | ⭐⭐ |
| [YouTube 内容流水线](usecases/youtube-content-pipeline.md) | 为 YouTube 频道自动化视频创意发掘、研究和追踪 | ⭐⭐⭐ |
| [多智能体内容工厂](usecases/content-factory.md) | 在 Discord 中运行研究、写作、设计三个智能体组成的内容流水线 | ⭐⭐⭐ |

## 基础设施与 DevOps

> 让服务器自己修自己，你只管睡觉

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [n8n 工作流编排](usecases/n8n-workflow-orchestration.md) | 通过 webhook 将 API 调用委托给 n8n 工作流，智能体从不接触凭证 | ⭐⭐⭐ |
| [自愈家庭服务器](usecases/self-healing-home-server.md) | 运行始终在线的基础设施智能体，自动发现并修复故障 | ⭐⭐⭐ |

## 生产力

> 把重复性工作交给 AI，专注于真正重要的事

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [定制早间简报](usecases/custom-morning-brief.md) | 每天醒来就有专属简报——新闻、任务、内容草稿、AI 推荐操作（国内适配） | ⭐ |
| [收件箱整理](usecases/inbox-declutter.md) | 自动总结新闻通讯并发送摘要邮件，告别邮件堆积 | ⭐ |
| [第二大脑](usecases/second-brain.md) | 随手发消息记住一切，在自定义仪表板中随时搜索 | ⭐ |
| [个人 CRM](usecases/personal-crm.md) | 自动从邮件和日历中发现并追踪联系人，支持自然语言查询 | ⭐⭐ |
| [健康与症状追踪器](usecases/health-symptom-tracker.md) | 追踪食物摄入和症状以识别过敏诱因，带有定期签到提醒 | ⭐ |
| [基于电话的个人助理](usecases/phone-based-personal-assistant.md) | 通过电话或短信从任何手机访问你的 AI 智能体 | ⭐⭐ |
| [多渠道个人助理](usecases/multi-channel-assistant.md) | 一个 AI 助理统管 Telegram、Slack、邮件和日历 | ⭐⭐ |
| [家庭日历与家务助理](usecases/family-calendar-household-assistant.md) | 聚合所有家庭日历到早间简报，监控消息获取预约，管理家庭库存 | ⭐⭐ |
| [Todoist 任务管理器](usecases/todoist-task-manager.md) | 将 AI 推理和进度日志同步到 Todoist，实时看到智能体在做什么 | ⭐⭐ |
| [多渠道 AI 客户服务](usecases/multi-channel-customer-service.md) | WhatsApp + Instagram + 邮件 + Google 评价统一到 AI 收件箱（国内适配） | ⭐⭐⭐ |
| [活动嘉宾确认](usecases/event-guest-confirmation.md) | 自动逐一呼叫嘉宾确认出席、收集备注并编译摘要 | ⭐⭐ |
| [项目状态管理](usecases/project-state-management.md) | 事件驱动的项目追踪，自动捕获上下文，取代静态看板 | ⭐⭐⭐ |
| [动态仪表板](usecases/dynamic-dashboard.md) | 实时仪表板，子智能体并行从 API、数据库和社交媒体获取数据 | ⭐⭐⭐ |
| [自主项目管理](usecases/autonomous-project-management.md) | 使用 STATE.yaml 模式协调多智能体项目，无需人工编排 | ⭐⭐⭐ |
| [多智能体专业团队](usecases/multi-agent-team.md) | 4 个专业 AI 智能体（战略+开发+营销+商务）作为你的虚拟团队 | ⭐⭐⭐ |

## 研究与学习

> 让 AI 帮你整理知识、追踪市场、发现机会

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [个人知识库 (RAG)](usecases/knowledge-base-rag.md) | 把 URL、推文和文章拖入聊天，构建可语义搜索的知识库 | ⭐⭐ |
| [AI 财报追踪器](usecases/earnings-tracker.md) | 追踪科技/AI 公司财报，自动化预览、提醒和详细摘要（国内适配） | ⭐⭐ |
| [语义记忆搜索](usecases/semantic-memory-search.md) | 为 OpenClaw 记忆文件添加向量驱动的语义搜索 | ⭐⭐ |
| [市场研究与产品工厂](usecases/market-research-product-factory.md) | 从 Reddit 和 X 挖掘真实痛点，让 AI 构建解决方案 MVP | ⭐⭐⭐ |
| [开发前创意验证器](usecases/pre-build-idea-validator.md) | 编码前自动扫描竞品，返回竞争度评分（国内适配） | ⭐⭐ |

## 金融与交易

> 用 AI 在预测市场上做模拟交易和策略回测

| 名称 | 描述 | 难度 |
|------|------|:---:|
| [Polymarket 自动驾驶](usecases/polymarket-autopilot.md) | 自动化预测市场模拟交易，带回测、策略分析和每日绩效报告 | ⭐⭐⭐ |



---

## 🌟 为什么选择这个合集？

| | 直接读英文 | 用翻译工具 | **本合集** |
|---|:---:|:---:|:---:|
| 中文母语阅读体验 | ❌ | ⚠️ 机翻生硬 | ✅ 原生中文内容 |
| 中国工具生态用例 | ❌ | ❌ | ✅ 飞书/小红书/淘宝等 |
| 新手入门指南 | ❌ | ❌ | ✅ 从零开始教 |
| 核心概念解释 | ❌ | ❌ | ✅ 通俗类比 |
| 难度标注 | ❌ | ❌ | ✅ 按能力选用例 |
| 真实场景验证 | ❌ | ❌ | ✅ 每个用例有来源佐证 |

---

## 🤝 贡献

我们欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解详细指南。

- 🆕 提交经过实际测试验证的用例
- 🐛 修正错误或过时信息

> 请只提交你已经使用过并验证有效的用例（至少使用一天）。我们重视真正让生活变得更好的真实想法！
>
> **注意：** 我们不接受与加密货币相关的用例。

---

## 📌 相关资源

- [OpenClaw 官方仓库](https://github.com/openclaw/openclaw) — 安装和官方文档
- [ClawHub 技能市场](https://clawhub.ai) — 浏览和安装技能

---

<div align="center">

**觉得有用？请给个 ⭐ Star 支持一下，让更多中文用户发现这个合集！**

</div>
