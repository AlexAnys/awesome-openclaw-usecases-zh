# Agent Usecase Contract（实验）

> 目的：让同一个用例可以被 OpenClaw、Hermes、Claude Code、Cursor 等不同个人智能体理解、筛选和安全执行。
>
> 这是**可选契约**。现有 `usecases/*.md` 不需要一次性迁移；没有契约时视为 `unverified`，不是“不兼容”。

## 为什么需要契约

当前用例已经按“痛点 → 所需技能 → 如何设置 → 实用建议”组织，适合人类阅读；但智能体执行时还需要提前知道：

1. 这个用例会不会写本地文件、发外部消息、公开发布？
2. 需要哪些账号、API Key、OAuth、网络环境？
3. 在 OpenClaw 之外，Hermes / Claude Code / Cursor 是否能跑？需要哪些适配？
4. 是否有最小验证命令和样例输出？

Contract 的目标是回答这些问题，让未来可以自动生成兼容矩阵、风险标签、E2E 检查和适配文档。

## 推荐 frontmatter

在用例文件顶部添加 YAML frontmatter，字段如下：

```yaml
---
id: cn-a-share-monitor
title: A 股每日行情监控
category: research-monitoring
language: zh
difficulty: 2

risk:
  - read_only
  - financial

agent_compat:
  openclaw: native
  hermes: partial
  claude_code: unverified
  cursor: unverified

requires:
  tools: [terminal, cron, python, send_message]
  skills: [akshare, mcp-cn-a-stock]
  accounts: []
  env: [TUSHARE_TOKEN]
  network: cn_preferred

delivery_targets: [feishu, dingtalk, wecom, telegram, slack, local]
cost: free_with_optional_paid

# verified 仅在真实运行后填写；evidence 必须指向运行日志或样例输出，
# 不能只指向 adapter 描述文档。
verified:
  last_run: null
  agent: null
  result: unverified
  evidence: null

test:
  level: sample_output
  command: "python -m venv /tmp/a-share-venv && /tmp/a-share-venv/bin/pip install akshare && /tmp/a-share-venv/bin/python -c 'import akshare as ak; print(ak.stock_market_activity_legu().head())'"
  expected_output_contains: ["上涨"]
---
```

> 字段名保持英文，便于脚本和 CI 解析；正文仍使用中文。

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | 用例 ID，建议与文件名一致（去掉 `.md`） |
| `category` | enum | 用例分类，见下方词表 |
| `difficulty` | 1/2/3 | 对应 README 中 ⭐ / ⭐⭐ / ⭐⭐⭐ |
| `risk` | list | 可能触达的风险/权限边界 |
| `agent_compat` | map | 不同智能体框架的适配状态 |
| `requires.tools` | list | 需要的通用工具能力，不绑定具体实现 |
| `requires.accounts` | list | 需要用户登录/授权的账号 |
| `requires.env` | list | 需要用户提供的环境变量 |
| `requires.network` | enum | 是否需要国内网络或特定出口 |
| `delivery_targets` | list | 可推送的目标渠道 |
| `verified` | map | 最近一次真实运行结果和证据链接；证据必须是运行日志、样例输出或测试报告 |
| `test` | map | 最小验证命令或 dry-run 说明 |

## 词表

### `category`

如果多个分类都适用，选择读者查找时最可能使用的主分类；例如 A 股监控优先放 `research-monitoring`，不要因为含金融数据就强制放 `finance`。

- `platform-bots`：飞书、钉钉、企业微信等平台机器人
- `content-publishing`：公众号、小红书、播客、社媒发布
- `research-monitoring`：研究、监控、竞品、论文、金融数据
- `office-cs`：办公自动化、会议、客服、电商
- `personal-assistant`：个人助理、早间简报、家庭/健康/CRM
- `infra-devops`：服务器、DevOps、可观测性、n8n
- `creative-build`：内容工厂、产品构建、Agent Swarm
- `finance`：金融、交易、市场数据
- `social-media`：Reddit、X、YouTube 等社交平台

### `risk`

| 标签 | 含义 |
|---|---|
| `read_only` | 只读信息收集、摘要、研究 |
| `writes_local` | 写入或移动本地文件 |
| `external_write` | 向第三方系统写入：发消息、建任务、改文档等 |
| `public_post` | 向公众渠道发布内容，如公众号、小红书、X |
| `credential_heavy` | 需要 OAuth / API Key / IP 白名单 / 高权限账号 |
| `financial` | 涉及金融市场、交易、投资信息 |
| `privacy` | 处理个人聊天记录、健康数据、联系人等隐私数据 |
| `regulated` | 医疗、金融、法律等强监管领域 |

### `agent_compat`

| 状态 | 含义 |
|---|---|
| `native` | 原生支持，照原用例即可运行 |
| `adapter` | 有适配文档，按差异说明运行 |
| `partial` | 部分可运行，受账号、平台、工具能力限制 |
| `unverified` | 未验证，不代表不可用 |
| `not_supported` | 明确不建议或无法支持 |

### `requires.tools`

- `terminal`：能执行 shell/CLI 命令
- `python`：能创建 venv / 执行 Python 脚本
- `file_io`：能读写/移动本地文件
- `cron`：能创建定时任务
- `web`：能搜索或抓取网页
- `browser`：能使用浏览器自动化
- `send_message`：能向 Slack/Telegram/Discord/本地等渠道推送
- `subagent`：能并行委派子智能体
- `mcp`：能连接 MCP 服务

### `requires.network`

- `any`：普通公网即可
- `cn_preferred`：国内出口更稳定，海外环境可能失败
- `cn_required`：实际需要国内账号/网络/服务器

## Adapter 文档约定

适配文档放在：

```text
adapters/<agent>/usecases/<id>.md
```

例如：

```text
adapters/hermes/usecases/cn-a-share-monitor.md
```

适配文档只写“与原用例不同的执行部分”，不复制完整用例。推荐结构：

```markdown
# Hermes 适配：<用例名>

> 这是 `../../../usecases/<id>.md` 的 Hermes 执行差异说明。

## 一句话适配结论
## 概念映射
## 前置安装差异
## 执行差异
## 最小验证
## 已知限制 / 风险
```

## 贡献原则

1. Contract 是为了降低执行风险，不是为了堆字段。
2. 没有实测证据时，`agent_compat` 应标为 `unverified` 或 `partial`。
3. 涉及公开发布、金融、医疗、隐私的用例必须保守标注风险。
4. 适配文档应短小；如果需要大段重写，说明原用例本身需要改进。
