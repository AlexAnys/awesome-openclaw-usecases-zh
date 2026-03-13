# 电商多 Agent 架构：从查数到全链路自动运营

> **技术要求分级**：单机版（飞书 + Skills）仅需基础命令行经验 ⭐⭐；SaaS（软件即服务）多租户部署需要 Kubernetes（容器编排平台）和云原生经验 ⭐⭐⭐。

电商卖家的日常运营散落在销售查询、库存监控、客服处理等多个场景。传统平台内置助手功能由平台预定义、渠道锁死在后台、只能被动等你提问。AWS 中国团队用 OpenClaw 完成了一次完整的 PoC（概念验证）——用 4 个平台级 Skill（技能）覆盖核心场景，多 Agent（智能体）绑定不同飞书群实现角色分工，Cron（定时任务）主动推送运营洞察，将能力构建从"编译型"变成"解释型"——写了就能用，改了就生效。

## 它能做什么

- **统一数据入口**：一个对话窗口覆盖销售、库存、客户、商品所有数据——"今天卖了多少？"直接回答，不用切换到后台
- **Skill 即能力**：用 Markdown（标记语言）文件定义数据查询逻辑，运营人员会写 `curl + jq` 即可封装，分钟级上线
- **多 Agent 角色分工**：销售助手绑定运营群、售后客服绑定客服群，各自只加载必要的 Skills 和人格，互不干扰
- **主动通知**：库存快断货、退货率异常飙升——AI 通过 Cron 主动告诉你，不用你去问
- **多渠道嵌入**：飞书、Slack、WebChat、Telegram 共享同一套 AI 引擎和 Skills
- **成本可控**：简单查询约 $0.08/次，复杂分析约 $0.12/次（PoC 实测）

## 所需技能

| 技能 | 来源 | 用途 |
|------|------|------|
| sales-query | 自建 Skill | 销售概况、订单列表、退货分析、趋势图 |
| product-ranking | 自建 Skill | 热销榜、滞销品、新品表现、品类分布 |
| inventory-alert | 自建 Skill | 低库存预警、断货提醒、补货建议 |
| customer-insight | 自建 Skill | 客户概览、新客/回头客分析、地域分布 |
| 飞书渠道 | 内置 Channel（渠道） | 多群聊绑定、WebSocket（长连接）模式 |
| Cron 定时任务 | 内置功能 | 库存预警、每日早报、异常监控 |

> 4 个 Skill 需要根据你的电商平台 API 自行开发。AWS PoC 提供了基于 Mock API（模拟接口）的完整参考实现，包含 12 个 SKU（库存单位）、50 个客户、20+ 个端点。

## 如何设置

### 1. 多 Agent 绑定配置

在 `openclaw.json` 中定义多个 Agent，每个通过 Binding（路由绑定）映射到对应的飞书群：

```json
{
  "agents": {
    "list": [
      { "id": "sales", "name": "销售助手", "workspace": "/data/ws/sales" },
      { "id": "support", "name": "售后客服", "workspace": "/data/ws/support" }
    ]
  },
  "bindings": [
    {
      "match": { "channel": "feishu", "peer": { "kind": "group", "id": "oc_abc111" } },
      "agentId": "sales"
    },
    {
      "match": { "channel": "feishu", "peer": { "kind": "group", "id": "oc_abc222" } },
      "agentId": "support"
    }
  ],
  "session": { "dmScope": "per-peer" }
}
```

> `dmScope: per-peer` 表示每个用户（按 `senderOpenId`）拥有独立的 Session（会话），对话历史互相隔离。

**架构概览：**

| 群/渠道 | 绑定 Agent | 加载的 Skills | 人格（SOUL.md） |
|---------|-----------|--------------|----------------|
| 飞书 - 销售运营群 | sales | sales-query、product-ranking | 数据优先、结论先行 |
| 飞书 - 售后客服群 | support | 退款处理、工单查询 | 耐心专业、安抚为主 |
| 飞书 - 私聊 | main（默认） | 全部 Skills | 通用助手 |

每个 Agent 拥有独立的 Workspace（工作区）目录：

```
workspace-{agentId}/
├── AGENTS.md      ← 行为规则
├── SOUL.md        ← 人格定义
├── IDENTITY.md    ← 身份信息
├── TOOLS.md       ← 工具权限
├── MEMORY.md      ← 持久记忆
└── skills/
    ├── sales-query.SKILL.md
    └── order-lookup.SKILL.md
```

### 2. SOUL.md 人格定义

**售后客服人格模板：**

```markdown
# 售后客服

你是电商团队的售后处理助手。

## 风格
- 耐心，不急躁
- 先确认问题，再给方案
- 涉及退款金额时给出明确数字和操作步骤

## 边界
- 只处理售后相关：退款、换货、工单、客诉
- 超过授权金额的退款，提醒用户找主管审批
```

**销售助手人格模板：**

```markdown
# 销售助手

你是电商团队的数据分析助手。

## 风格
- 数据优先，结论先行
- 主动给出同比/环比对比
- 数据异常时主动标注并给出可能原因

## 边界
- 只处理销售和商品相关查询
- 不执行退款、改价等写操作
```

### 3. 平台级 Skill 示例

Skill 文件是 Markdown 文档，描述数据查询逻辑。以下是 `sales-query` Skill 的示例：

以下提示词定义了销售查询 Skill 的行为和输出格式（因面向中文用户，Skill 内容使用中文编写）：

```markdown
# sales-query

查询销售数据。支持按日期范围、商品类别、地域等维度筛选。

## 工具

使用 exec 工具执行以下命令获取数据：

### 今日销售概况

curl -s -H "Authorization: Bearer $API_TOKEN" \
  "$API_ENDPOINT/v1/orders/stats?date=$(date +%Y-%m-%d)" | jq '.'

### 指定日期范围的订单列表

curl -s -H "Authorization: Bearer $API_TOKEN" \
  "$API_ENDPOINT/v1/orders?start_date={start}&end_date={end}&page=1" | jq '.orders'

## 输出格式
- 金额使用 ¥ 符号，保留两位小数
- 百分比变化标注 ↑ 或 ↓
- 超过 10 条数据时使用表格展示
```

> **关键洞察**：Skill 的能力边界由 API 决定——只要有 API，就可以封装成 Skill。从"工程师写代码、定义 Function Schema、测试发版"变成"运营人员写 Markdown、放到 skills 目录、立即生效"。

### 4. Cron 定时库存预警

以下配置实现每 30 分钟自动检查库存并推送预警：

```json
{
  "kind": "every",
  "interval": "30m",
  "prompt": "检查库存状况，如果有断货或低库存商品，列出详情和补货建议",
  "session": "isolated",
  "delivery": { "channel": "last" }
}
```

更多定时场景：

| 场景 | 调度配置 | 用途 |
|------|---------|------|
| 每日早报 | `cron: "0 8 * * *"` | 昨日销售概况 + 异常订单 + 补货提醒 |
| 实时监控 | `every: "15m"` | 大额退款告警 |
| 周报 | `cron: "0 9 * * 1"` | TOP 10 热销品 + 滞销品分析 |

### 5. 安全边界配置

电商场景的安全控制尤其重要——AI 能查数据，但不能随意退款改价。

**工具执行白名单（safeBins）：**

```json
{
  "tools": {
    "exec": {
      "host": "node",
      "security": "allowlist",
      "ask": "on-miss",
      "safeBins": ["jq", "grep", "curl", "cut", "sort", "uniq", "head", "tail", "tr", "wc"]
    }
  }
}
```

**Skill 级别开关：**

```json
{
  "skills": {
    "entries": {
      "query-orders": { "enabled": true },
      "query-inventory": { "enabled": true },
      "refund": { "enabled": false },
      "change-price": { "enabled": false }
    }
  }
}
```

> **重要**：`safeBins` 在可执行文件级别操作，无法区分 `curl GET`（查询）和 `curl POST`（退款）。因此**写操作的审批是业务 API 的职责，不是 AI 层的职责**——必须在 API 侧引入人工确认步骤。

**凭证管理——环境变量注入：**

```bash
# API 凭证通过环境变量注入，绝不硬编码在脚本或配置文件中
export API_TOKEN="your-api-token"
export API_ENDPOINT="https://api.your-platform.com"
```

### 6. 飞书渠道配置

OpenClaw 使用 WebSocket 模式连接飞书，不需要公网入站端口：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxxxx",
      "appSecret": "xxxxx",
      "dmPolicy": "open",
      "groupPolicy": "open",
      "requireMention": true
    }
  }
}
```

**配置步骤：**

1. 在[飞书开放平台](https://open.feishu.cn/)创建企业自建应用，获取 App ID 和 App Secret
2. 配置权限：获取私聊/群聊消息、获取群信息、发送消息等
3. 启用 Bot 能力
4. 将凭证写入 `openclaw.json`
5. 启动 OpenClaw 实例——它会主动向飞书发起 WebSocket 连接
6. 回到飞书开放平台配置事件订阅并发布应用

> **注意顺序**：事件订阅必须在 WebSocket 连接成功之后配置，否则飞书会报错。

## 效果与成本

### PoC 实测数据

| 指标 | 数值 |
|------|------|
| 简单查询（"今天卖了多少"） | ~18K input token（词元） + 1.7K output token ≈ **$0.08** |
| 复杂分析（"本周退货趋势 + 原因分析"） | ~22K input + 3K output ≈ **$0.12** |
| 基础设施（100 商家分摊） | EKS + EFS（见下方 SaaS 部分）≈ **$17.74/商家/月** |

### 成本优化手段

- **Prompt Cache（提示词缓存）**：Bedrock（AWS 基础模型服务）的缓存可节省 80-90% 的 input token 费用
- **模型分级**：简单查询用 Claude Haiku（成本约为 Sonnet 的 1/10），复杂分析才用 Sonnet
- **优化后目标**：约 $45/商家/月（中等使用频率）

### SaaS 多租户部署成本（进阶）

AWS PoC 验证了 Per-tenant Pod（每租户独立容器组）方案，使用 EKS（弹性容器服务） + EFS（弹性文件系统） + Envoy Gateway（API 网关）路由：

| 规模 | 节点配置 | 合计/月 | 分摊/商家/月 |
|------|---------|---------|-------------|
| 10 商家 | 2x r7i.large | ~$190 | ~$19.0 |
| 100 商家 | 5x r7i.xlarge | ~$656 | ~$6.6 |
| 1000 商家 | 24x r7i.2xlarge | ~$5,671 | ~$5.7 |

> 完整的 K8s（Kubernetes 简称）部署方案（Envoy Gateway 配置、租户管理脚本、安全隔离细节）请参考 [AWS 原文](https://aws.amazon.com/cn/blogs/china/exploring-openclaw-use-cases-in-ecommerce-platforms/) 和 [GitHub 仓库](https://github.com/SharonNi/jdopenclaw)。

## 实用建议

- **写操作必须有人工确认**：退款、改价等操作在 API 侧加审批流程，`safeBins` 无法区分读写——一旦 `curl` 进了白名单，所有 curl 调用都会被自动批准
- **从只读开始**：先接入查询类 API（销售、库存、客户），跑通后再渐进覆盖操作类场景
- **人格隔离很重要**：售后客服"语气温和、优先安抚" vs 销售助手"数据优先、结论先行"——SOUL.md 的差异直接影响用户体验
- **Skill 写法门槛很低**：只要会写 `curl + jq`，就能封装成 Skill，不需要工程师介入
- **PoC 使用了 Mock API**：AWS 的 PoC 基于模拟数据（12 个 SKU、50 个客户、20+ 个端点），接入真实平台 API 需要自行适配接口格式
- **飞书事件订阅顺序**：先启动 OpenClaw 建立 WebSocket 连接，再在飞书开放平台配置事件订阅，否则会报错

## 延伸场景（社区探索）

以下场景在社区中有讨论和初步实践，但尚未达到完整的端到端验证标准，供参考：

### 竞品价格监控

安装 `browserwing`（浏览器自动化 Skill）+ `jd-auto-order` + `im-master`（跨平台消息），对 OpenClaw 说"监控京东竞品价格变化，降价超过 10% 立即通知我"即可运行。BrowserWing 支持淘宝/京东等国内网站，支持录制脚本模式降低 Token 消耗。

> **注意**：V2EX 用户反馈该插件"不太稳定，经常断开需手工重连"，建议在生产环境中增加重连机制。

### 跨境电商多角色协作

社区博主实测了 5 个 AI 数字员工方案：VOC（客户之声）市场调研员、GEO（地理优化）内容优化师、Reddit 种草手、TikTok 视频生成师、数据分析师，通过飞书路由实现角色分工。具体实施细节和效果数据有待更多独立验证。

## 出处

由 AWS 中国团队（倪惠青、黄筱婷）完成的 PoC 项目，使用 OpenClaw 构建电商卖家助手，验证了 Skill 机制、多 Agent 路由、Cron 主动通知和 SaaS 多租户部署的可行性。PoC 代码开源在 GitHub。

## 相关链接

- [AWS 中国博客 — OpenClaw 在电商平台的应用场景探索](https://aws.amazon.com/cn/blogs/china/exploring-openclaw-use-cases-in-ecommerce-platforms/) — 原始完整文章
- [jdopenclaw GitHub 仓库](https://github.com/SharonNi/jdopenclaw) — PoC 完整代码（Mock API、K8s 配置、4 个 Skills、租户管理脚本）
- [飞书开放平台](https://open.feishu.cn/) — 创建企业自建应用
- [BrowserWing](https://github.com/nicepkg/browserwing) — 浏览器自动化 Skill（社区）
