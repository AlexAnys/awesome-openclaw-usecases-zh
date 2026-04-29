# Hermes 第一批适配计划

> 目标：在保留 OpenClaw 原生用例的前提下，补充 Hermes 执行差异说明，让读者更容易判断“我的智能体能不能跑”。

## 筛选标准

| 维度 | 权重 | 说明 |
|---|---:|---|
| Hermes 工具匹配度 | 高 | 是否能用 Hermes 现有 terminal / cron / file / send_message / subagent 跑通 |
| 低风险可复现 | 高 | 是否能在没有生产账号的情况下做最小验证 |
| 中国本土价值 | 高 | 是否体现飞书、中文互联网、A 股、国内办公等差异化场景 |
| 传播价值 | 中 | 是否容易被读者理解为“个人智能体真的有用” |
| 凭证/风控复杂度 | 负向 | OAuth、IP 白名单、公开发布、账号封禁风险越高，越不适合第一批 |

## 第一批用例

| # | 用例 | Hermes 状态 | 风险标签 | 为什么入选 |
|---|---|---|---|---|
| 1 | [办公自动化套件](../../usecases/cn-office-automation.md) | `native` | `writes_local` | 最适合做 5 分钟入门：本地文件整理可无账号跑通，能展示 Hermes 文件/终端能力 |
| 2 | [自定义早间简报](../../usecases/custom-morning-brief.md) | `adapter` | `read_only`, `external_write` | 展示 cron + web + delivery，是个人智能体最容易被理解的场景 |
| 3 | [中文互联网 30 天研究](../../usecases/cn-internet-research-30days.md) | `adapter` | `read_only` | 中国本土差异化最强，适合做“中文互联网研究员”传播案例 |
| 4 | [A 股每日行情监控](../../usecases/cn-a-share-monitor.md) | `partial` | `read_only`, `financial` | 展示定时金融信息整理，但必须明确非投资建议和数据源 fallback |
| 5 | [飞书全能操作台（Lark CLI）](../../usecases/cn-feishu-lark-cli.md) | `partial` | `credential_heavy`, `external_write` | 国内办公生态强钩子；安装可验证，生产使用需要 OAuth 和 dry-run |
| 6 | [会议纪要与待办自动化](../../usecases/meeting-notes-action-items.md) | `partial` | `privacy`, `external_write` | 高价值办公场景；文本纪要本地可跑，平台拉取/创建任务需要账号适配 |

## 暂缓用例

| 用例 | 暂缓原因 |
|---|---|
| 微信公众号自动发布 | `public_post` + AppID/AppSecret + IP 白名单 + 内容审核；应等 dry-run/confirm 模式稳定后再适配 |
| 小红书内容自动化 | RPA/反爬/账号风控较高，不适合作为第一批入门适配 |
| 数字人格蒸馏 | 隐私和 PIPL 风险高，需要先补隐私审查模板 |
| 多智能体协作操作系统 / Agent Swarm | Hermes 和 OpenClaw 的子智能体语义差异较大，需要单独设计 |
| 多渠道客服 / 电商多 Agent | 依赖真实业务系统和账号权限，不适合第一批复现 |

## 后续工作

- 给更多用例逐步补充可选 frontmatter，而不是一次性迁移全部文件。
- 为低风险用例增加 `tests/<usecase>/run.sh` 和样例输出，优先覆盖：办公自动化、早间简报、中文互联网研究、A 股信息整理。
- 等 Hermes adapter 模式稳定后，再接受 Claude Code / Cursor / OpenHands 等其他框架的适配文档。
- 公开发布、金融、医疗、隐私类用例先补风险模板，再补自动化执行说明。
