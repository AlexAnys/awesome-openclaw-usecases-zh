# Hermes Adapter（实验）

> 目标：让 OpenClaw 用例可以被 Hermes 用户安全、可复现地执行，同时保留原仓库的 OpenClaw 原生定位。

Hermes 与 OpenClaw 都是个人智能体，但执行层不同：工具名称、技能目录、定时任务、消息渠道、权限确认方式都不完全一样。因此这里采用 **diff-only adapter**：原用例仍在 `usecases/`，本目录只写 Hermes 下的差异。

## 什么时候需要看本目录

- 你想用 Hermes 跑本仓库的 OpenClaw 用例；
- 用例涉及定时任务、消息推送、CLI、OAuth、公开发布、金融/隐私风险；
- 你想为其他 agent 框架贡献适配文档。

## OpenClaw → Hermes 概念映射

| OpenClaw 概念 | Hermes 对应能力 | 说明 |
|---|---|---|
| Skill / ClawHub | Hermes Skill / 外部 CLI / MCP | 方法可复用，但安装路径与调用方式不同 |
| Channel | Slack / Telegram / Discord / local delivery / `send_message` | 飞书、钉钉、企微通常需要 CLI、Webhook 或额外 adapter |
| Cron Job | Hermes `cronjob` / 平台定时任务 | 必须显式写时区，推荐 `Asia/Shanghai` |
| Workspace | Hermes 当前工作目录 / 项目目录 | 不要硬编码 `~/.agents`、`~/.openclaw` 等路径 |
| SOUL.md / AGENTS.md | Hermes SOUL / AGENTS / skills | 语义类似，文件位置和加载机制不同 |
| Sub-agent | Hermes `delegate_task` / coding agent skills | 适合研究、代码、审查并行任务 |
| MCP | Hermes MCP / native tools | 可复用协议，但配置位置不同 |

## Hermes 执行原则

1. **先 dry-run，再写入。** 涉及发消息、改文档、发公众号、小红书、邮件等外部写操作时，默认只预演。
2. **凭证不进仓库。** API Key、OAuth token、邮箱授权码只能放环境变量或本地私有配置。
3. **公共发布必须人工确认。** 公众号、小红书、X、抖音等场景必须保留人审。
4. **金融/医疗/隐私场景只做整理，不做决策。** A 股、健康、人格蒸馏等用例必须明确边界。
5. **国内平台要标注网络和权限限制。** 飞书 OAuth、微信 IP 白名单、AKShare 东方财富接口、小红书反爬都可能导致“装好了但跑不通”。

## 适配状态

| 状态 | 含义 |
|---|---|
| `native` | Hermes 现有工具可直接完成 |
| `adapter` | 按本目录说明可完成主要流程 |
| `partial` | 只能跑安装/样例/dry-run，生产层依赖账号或外部权限 |
| `unverified` | 尚未验证 |
| `not_supported` | 不建议在 Hermes 下执行 |

## 第一批适配用例

详见 [FIRST-BATCH.md](./FIRST-BATCH.md)。当前优先选择：低风险、能展示 Hermes 核心能力、同时有中国本土场景代表性的用例。

| 用例 | 状态 | 入口 |
|---|---|---|
| 办公自动化套件 | `native` | [cn-office-automation](./usecases/cn-office-automation.md) |
| 自定义早间简报 | `adapter` | [custom-morning-brief](./usecases/custom-morning-brief.md) |
| 中文互联网 30 天研究 | `adapter` | [cn-internet-research-30days](./usecases/cn-internet-research-30days.md) |
| A 股每日行情监控 | `partial` | [cn-a-share-monitor](./usecases/cn-a-share-monitor.md) |
| 飞书 Lark CLI | `partial` | [cn-feishu-lark-cli](./usecases/cn-feishu-lark-cli.md) |
| 会议纪要与待办自动化 | `partial` | [meeting-notes-action-items](./usecases/meeting-notes-action-items.md) |

## 适配文档格式

每个文件只回答四件事：

1. Hermes 下能跑到什么程度？
2. 原用例的哪些步骤要替换？
3. 最小验证怎么做？
4. 风险和限制是什么？

如果需要完整功能描述、痛点、提示词，请回到对应的 `usecases/*.md`。
