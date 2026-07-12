# INDEX.md

50 个用例的扁平索引。每行 = 路径 / 一句话中文摘要 / 风险标签。Agent 可用本文件做仓库导航，无需 grep 文件名。

> 分组按内容归属，**每个用例在本文件只出现一次**（深度国内适配的国际用例归入「中国特色」组，不再重复列于通用组）；README 的人类目录会把这类用例双列展示，因此两边的分节计数不同、总数一致（50）。

## 风险标签

- `read-only` — 仅读取数据（不写本地、不发外部）
- `writes-local` — 写入本地文件 / 工作区（无外发）
- `external-api` — 调用外部 API（消耗配额或费用）
- `external-write` — 写入外部账户（发消息 / 推送 / 修改云端数据）
- `public-post` — 在公共平台发布内容（小红书 / 公众号 / X / YouTube 等）
- `credential-heavy` — 需要多个高敏感凭证或长期 token
- `privacy` — 处理个人聊天 / 健康 / 联系人等敏感数据
- `financial` — 涉及财务、行情或交易决策

执行任何 `external-*`、`public-post`、`financial` 标签的用例前，请按 [AGENTS.md](AGENTS.md) 的 Reading Protocol 先 dry-run 并取得用户确认。

---

## 中国特色用例（23）

### 平台机器人

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [cn-feishu-ai-assistant.md](usecases/cn-feishu-ai-assistant.md) | 把 OpenClaw 部署为飞书机器人，对话直接触发 AI 任务，含文档自动化 | `external-api` `external-write` |
| [cn-feishu-lark-cli.md](usecases/cn-feishu-lark-cli.md) | 让 Agent 以你的身份操作飞书，11 领域 200+ 命令 | `external-api` `external-write` `credential-heavy` |
| [cn-dingtalk-ai-assistant.md](usecases/cn-dingtalk-ai-assistant.md) | 把 OpenClaw 部署为钉钉机器人，Stream 模式无需公网 IP | `external-api` `external-write` |
| [cn-wecom-ai-assistant.md](usecases/cn-wecom-ai-assistant.md) | 企业微信中使用 AI，含个微插件方案 | `external-api` `external-write` |

### 内容创作与发布

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [cn-xiaohongshu-automation.md](usecases/cn-xiaohongshu-automation.md) | 小红书选题/文案/封面/定时发布全流程 | `public-post` `external-write` |
| [cn-wechat-mp-automation.md](usecases/cn-wechat-mp-automation.md) | 微信公众号 Markdown 排版 + 草稿箱推送，含权限矩阵与 IP 白名单 | `public-post` `external-write` `credential-heavy` |
| [podcast-production-pipeline.md](usecases/podcast-production-pipeline.md) | 播客选题→剪辑→发布全流程（小宇宙 / 喜马拉雅 / B 站适配） | `public-post` `external-write` |

### 数据研究与监控

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [cn-a-share-monitor.md](usecases/cn-a-share-monitor.md) | A 股盘前简报 + 盘后复盘 + 板块资金流向（AKShare / MCP） | `external-api` `financial` |
| [earnings-tracker.md](usecases/earnings-tracker.md) | 财报追踪与提醒（含 A 股 AKShare 适配，业绩预告 / 快报自动化） | `external-api` `financial` |
| [competitive-intelligence.md](usecases/competitive-intelligence.md) | 竞品周报（Perplexity + Firecrawl + 百度指数 / 微信指数 / 飞书推送） | `external-api` |
| [pre-build-idea-validator.md](usecases/pre-build-idea-validator.md) | 编码前竞品扫描（百度指数 / 微信指数 / V2EX / 少数派） | `external-api` |
| [cn-internet-research-30days.md](usecases/cn-internet-research-30days.md) | 8 大中文平台 30 天内容研究，三级降级，零配置可用 4 个免费源 | `external-api` |
| [hf-papers-research-discovery.md](usecases/hf-papers-research-discovery.md) | 每日 HuggingFace 热门 ML 论文筛选 + arXiv 深读（HF 镜像 / 飞书推送） | `external-api` |
| [arxiv-paper-reader-latex-writer.md](usecases/arxiv-paper-reader-latex-writer.md) | arXiv 论文获取 / 章节浏览 / 摘要 + LaTeX 即时编译（中文模板适配） | `external-api` `writes-local` |

### 办公与客户服务

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [cn-office-automation.md](usecases/cn-office-automation.md) | 邮件 / 文件 / 会议纪要 / 周报，支持 163 / QQ / Outlook | `external-api` `external-write` `credential-heavy` |
| [meeting-notes-action-items.md](usecases/meeting-notes-action-items.md) | 会议转录→纪要→任务（飞书妙记 / 腾讯会议 / 钉钉） | `external-api` `external-write` |
| [multi-channel-customer-service.md](usecases/multi-channel-customer-service.md) | 多渠道客服（企微 / 抖音 / 小红书 / WhatsApp / Instagram） | `external-api` `external-write` `public-post` |
| [cn-ecommerce-multi-agent.md](usecases/cn-ecommerce-multi-agent.md) | 电商多 Agent（销售 / 库存 / 客户）+ 飞书群协作 | `external-api` `external-write` |

### 个人助理与智能体架构

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [custom-morning-brief.md](usecases/custom-morning-brief.md) | 每日定时早间简报到飞书 / 钉钉，支持中文新闻源 | `external-api` `external-write` |
| [digital-persona-distillation.md](usecases/digital-persona-distillation.md) | 12+ 平台聊天记录提取 4 维人格档案，含 PIPL 合规提醒 | `privacy` `credential-heavy` |
| [cn-multi-agent-operating-system.md](usecases/cn-multi-agent-operating-system.md) | OpenClaw 多智能体协作 OS（专业分工与稳定迭代） | `writes-local` |
| [agent-swarm-dev-team.md](usecases/agent-swarm-dev-team.md) | OpenClaw 编排 Codex + Claude Code 舰队的全自动开发流水线 | `writes-local` `external-api` |
| [multica-managed-agents.md](usecases/multica-managed-agents.md) | 把 OpenClaw / Claude Code / Codex / Hermes 拉进同一 Web 看板，Issue 即任务，Apache 2.0 自部署 | `writes-local` `external-api` `credential-heavy` |

---

## 通用场景（27）

### 社交媒体

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [daily-reddit-digest.md](usecases/daily-reddit-digest.md) | 偏好驱动的每日 subreddit 精选摘要 | `external-api` |
| [daily-youtube-digest.md](usecases/daily-youtube-digest.md) | 关注频道的每日新视频摘要 | `external-api` |
| [x-account-analysis.md](usecases/x-account-analysis.md) | 你的 X（Twitter）账号定性分析报告 | `external-api` |
| [multi-source-tech-news-digest.md](usecases/multi-source-tech-news-digest.md) | 109+ 来源科技新闻聚合，含质量评分与多渠道分发 | `external-api` `external-write` |

### 创意与构建

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [overnight-mini-app-builder.md](usecases/overnight-mini-app-builder.md) | 目标驱动自主任务，一夜造迷你应用 | `writes-local` `external-api` |
| [youtube-content-pipeline.md](usecases/youtube-content-pipeline.md) | YouTube 频道创意发掘 / 研究 / 追踪 | `external-api` `public-post` |
| [content-factory.md](usecases/content-factory.md) | Discord 中研究 + 写作 + 设计三 Agent 内容流水线 | `external-api` `external-write` |

### 基础设施与 DevOps

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [n8n-workflow-orchestration.md](usecases/n8n-workflow-orchestration.md) | webhook 委托 n8n 工作流，Agent 不接触凭证 | `external-api` `credential-heavy` |
| [opik-openclaw-observability.md](usecases/opik-openclaw-observability.md) | Opik 接入 OpenClaw 链路追踪，监控 token / 成本 | `external-api` `writes-local` |
| [self-healing-home-server.md](usecases/self-healing-home-server.md) | 始终在线的基础设施 Agent，自动发现并修复故障 | `credential-heavy` `writes-local` |

### 生产力

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [inbox-declutter.md](usecases/inbox-declutter.md) | 自动总结 newsletter 并发摘要邮件 | `external-api` `external-write` |
| [second-brain.md](usecases/second-brain.md) | 随手发消息记录一切，自定义仪表板搜索 | `writes-local` |
| [personal-crm.md](usecases/personal-crm.md) | 自动从邮件 / 日历发现并追踪联系人，支持自然语言查询 | `external-api` `privacy` |
| [health-symptom-tracker.md](usecases/health-symptom-tracker.md) | 食物 / 症状追踪，识别过敏诱因 | `privacy` `writes-local` |
| [phone-based-personal-assistant.md](usecases/phone-based-personal-assistant.md) | 通过电话或短信访问你的 AI 智能体 | `external-api` `credential-heavy` |
| [multi-channel-assistant.md](usecases/multi-channel-assistant.md) | 一个助理统管 Telegram / Slack / 邮件 / 日历 | `external-api` `external-write` |
| [family-calendar-household-assistant.md](usecases/family-calendar-household-assistant.md) | 聚合家庭日历到早间简报 + 库存管理 | `external-api` `privacy` |
| [todoist-task-manager.md](usecases/todoist-task-manager.md) | AI 推理与进度日志同步到 Todoist | `external-api` `external-write` |
| [event-guest-confirmation.md](usecases/event-guest-confirmation.md) | 自动逐一呼叫嘉宾确认出席并编译摘要 | `external-write` |
| [project-state-management.md](usecases/project-state-management.md) | 事件驱动项目追踪，取代静态看板 | `writes-local` |
| [dynamic-dashboard.md](usecases/dynamic-dashboard.md) | 实时仪表板，子智能体并行从 API / 数据库 / 社交媒体取数 | `external-api` |
| [autonomous-project-management.md](usecases/autonomous-project-management.md) | STATE.yaml 模式协调多智能体项目，无需人工编排 | `writes-local` |
| [multi-agent-team.md](usecases/multi-agent-team.md) | 4 个专业 AI 智能体（战略 + 开发 + 营销 + 商务） | `external-api` |

### 研究与学习

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [knowledge-base-rag.md](usecases/knowledge-base-rag.md) | URL / 推文 / 文章语义搜索知识库 | `writes-local` `external-api` |
| [semantic-memory-search.md](usecases/semantic-memory-search.md) | 为 OpenClaw 记忆文件加向量驱动语义搜索 | `writes-local` |
| [market-research-product-factory.md](usecases/market-research-product-factory.md) | Reddit / X 挖痛点 + AI 构建解决方案 MVP | `external-api` `writes-local` |

### 金融与交易

| 用例 | 摘要 | 风险标签 |
|---|---|---|
| [polymarket-autopilot.md](usecases/polymarket-autopilot.md) | Polymarket 模拟交易 + 回测 + 每日绩效报告 | `financial` `external-api` |
