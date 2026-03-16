# 自由职业者一站式助手

> 含国内适配：国内发票/报税工具现状说明

接活、交付、开票、催款、报税——自由职业者每天在十几个工具间切换，真正用于交付的时间不到一半。你需要一个统一的 AI 助手把这些琐事串起来，让你专注在高价值工作上。

这个工作流将自由职业者的核心运营整合到 OpenClaw 中：

- **客户管理**：自动记录沟通历史，会议前生成客户简报
- **时间追踪**：通过 Toggl 或 Kimai 记录工时，识别未计费时间
- **发票与催款**：连接 FreshBooks 生成发票，自动发送逾期提醒
- **费用分类**：按税务类别归档支出，为报税季做准备
- **排期管理**：Google Calendar 集成，跨项目排期防冲突
- **方案/报价单**：根据历史项目数据生成报价，内置范围边界条款

## 它能做什么

| 功能 | 说明 | 难度 |
|------|------|------|
| 客户 CRM（客户关系管理） | 扫描邮件/日历，自动建立客户档案和互动时间线 | ⭐ |
| 工时记录 | 按项目/客户分类计时，标记未计费时间 | ⭐ |
| 发票生成 | 基于工时和里程碑自动生成发票并发送 | ⭐⭐ |
| 逾期催款 | 阶梯式催款（3天/7天/14天），语气逐步正式 | ⭐ |
| 费用归档 | OCR（光学字符识别）识别票据，按税务类别自动分类 | ⭐⭐ |
| 排期防冲突 | 跨项目日历视图，交付日期预警 | ⭐ |
| 报价单生成 | 参考历史项目生成报价，含范围边界和修改次数限制 | ⭐⭐ |
| 范围蔓延检测 | 对比原始 SOW（工作说明书），标记超出范围的请求 | ⭐⭐⭐ |

## 所需技能

核心技能（按功能模块）：

| 技能 | 用途 | 安装方式 |
|------|------|---------|
| `gog` CLI | Gmail + Google Calendar 集成 | [gog 安装指南](https://github.com/obot-platform/gog) |
| `kimai-time-tracking` | 工时记录与项目计时 | 通过 OpenClaw Skills Registry 安装 |
| `freshbooks` | 发票生成、客户管理、账款追踪 | 通过 OpenClaw Skills Registry 安装 |
| `google-calendar-integration` | 排期管理、空闲时段查询 | 通过 OpenClaw Skills Registry 安装 |
| `web_search`（内置） | 市场费率调研、客户背景研究 | 无需安装 |

可选技能：

- `xero` — 如果你用 Xero 而非 FreshBooks 做会计
- `fiken` — 北欧用户的会计方案（支持发票、联系人、时间追踪）
- Toggl 集成 — 如果你已有 Toggl 工时数据，可通过 Freelancer Command Center 接入

## 如何设置

### 1. 准备工作

确保以下环境变量已配置（建议写入 `.env` 文件）：

```bash
# FreshBooks OAuth（开放授权，发票和客户管理）
export FRESHBOOKS_CLIENT_ID="your_client_id"
export FRESHBOOKS_CLIENT_SECRET="your_client_secret"
export FRESHBOOKS_ACCOUNT_ID="your_account_id"

# Kimai（工时记录）
export KIMAI_BASE_URL="https://your-kimai-instance.com"
export KIMAI_API_TOKEN="your_api_token"

# Google Workspace（邮件和日历）
# 通过 gog auth 完成 OAuth，令牌保存在本地

# Telegram（通知推送）
export TELEGRAM_BOT_TOKEN="your_bot_token"
```

### 2. 安装技能

```bash
# 安装时间追踪技能
openclaw skills install kimai-time-tracking

# 安装发票技能
openclaw skills install freshbooks

# 安装日历技能
openclaw skills install google-calendar-integration

# 完成 Google 授权
gog auth
```

### 3. 创建客户数据库

让 OpenClaw 创建一个本地 SQLite 数据库来存储客户和项目信息：

```sql
-- 客户表
CREATE TABLE clients (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT,
  company TEXT,
  rate_type TEXT,        -- 'hourly' 或 'project'
  default_rate REAL,
  payment_terms INTEGER, -- 付款期限（天）
  first_contact TEXT,
  last_contact TEXT,
  notes TEXT
);

-- 项目表
CREATE TABLE projects (
  id INTEGER PRIMARY KEY,
  client_id INTEGER REFERENCES clients(id),
  name TEXT NOT NULL,
  status TEXT,           -- 'active', 'completed', 'paused'
  budget REAL,
  scope_doc TEXT,        -- 原始 SOW 摘要
  start_date TEXT,
  deadline TEXT,
  billed_total REAL DEFAULT 0
);
```

### 4. 配置主提示词

以下提示词将 OpenClaw 配置为你的全流程业务助手：

```text
You are my freelance business assistant. You manage clients, time, invoices, and scheduling.

DAILY ROUTINES (cron jobs):
1. 7 AM: Scan Gmail and Calendar for new client interactions. Update the clients database.
2. 7:30 AM: Check today's calendar. For each meeting, prepare a client brief from CRM data and email history. Post to Telegram "freelance" topic.
3. 6 PM: Log today's unbilled time entries from Kimai. Alert me if any project has >2 hours unbilled.
4. Friday 5 PM: Generate weekly summary — hours worked per client, invoices sent, payments received, upcoming deadlines.

CLIENT MANAGEMENT:
- When I say "new client [name] [email] [rate]", create a client record and set up a project folder.
- When I say "prep for [client name]", search CRM + email history and give me a briefing.
- Track all email threads per client automatically.

TIME TRACKING:
- When I say "start [project name]", begin a Kimai timer for that project.
- When I say "stop", end the current timer.
- At end of day, categorize unbilled time and suggest which entries to bill.

INVOICING:
- When I say "invoice [client name]", pull unbilled Kimai hours + any fixed milestones, generate a FreshBooks invoice, and send it.
- Track payment status. Send reminders:
  - Day 3 overdue: friendly reminder
  - Day 7: firmer follow-up
  - Day 14: final notice with late fee mention
- When I say "cash flow", show outstanding invoices, expected payments this month, and projected revenue for next 30/60/90 days.

PROPOSALS:
- When I say "quote for [description]", draft a proposal based on similar past projects. Include:
  - Scope boundaries (what IS and IS NOT included)
  - Revision limits
  - Payment schedule (e.g., 50% upfront, 50% on delivery)
  - Timeline estimate

SCOPE MANAGEMENT:
- For active projects, compare client requests against the original scope doc.
- If a request falls outside scope, draft a polite change-order response with cost estimate.

SCHEDULING:
- When I say "schedule [task] by [date]", check calendar for conflicts and block time.
- Alert me 48 hours before any project deadline.
- When I say "availability this week", show open slots for new work.
```

### 5. 逐步测试

按以下顺序测试每个模块：

1. **客户管理**：添加一个测试客户，验证数据库写入
2. **时间追踪**：启动/停止计时器，确认 Kimai 记录正确
3. **日历**：创建一个测试事件，验证冲突检测
4. **发票**：生成一张测试发票（FreshBooks 沙箱模式）
5. **催款**：模拟一张逾期发票，验证提醒消息
6. **报价单**：让 OpenClaw 基于描述生成一份报价

## 实用建议

- **从单一模块开始**：不要试图一次配置所有功能。建议从时间追踪 + 发票开始，这是投入产出比最高的组合
- **SOW 模板化**：让 OpenClaw 帮你维护一个 SOW（工作说明书）模板库，新项目直接套用，范围蔓延检测才有基准
- **费率定期复盘**：每月让 OpenClaw 计算你的"有效时薪"（总收入 / 总工时），对比市场费率决定是否调价
- **催款语气库**：预先写好三档催款模板（温和/正式/最终通知），让 OpenClaw 按逾期天数自动选择
- **备份客户数据**：SQLite 数据库定期备份到云端，避免数据丢失

## 相关链接

- [OpenClaw Freelancer Command Center](https://popularaitools.ai/openclaw-freelancer-review/) — 10 个专为自由职业者设计的 AI 技能套件评测
- [Kimai Time Tracking 技能](https://tessl.io/registry/skills/github/openclaw/skills/kimai-time-tracking) — 开源工时记录集成
- [FreshBooks 技能](https://lobehub.com/skills/openclaw-skills-freshbooks) — 发票和客户管理
- [Fiken 会计技能](https://github.com/kristianvast/skill-fiken) — 支持发票、联系人、时间追踪
- [gog CLI](https://github.com/obot-platform/gog) — Google Workspace 集成
- [10 AI Agents for Freelancers](https://www.mindstudio.ai/blog/ai-agents-for-freelancers) — MindStudio 自由职业者 AI 代理方案总结
- [awesome-openclaw-skills](https://github.com/VoltAgent/awesome-openclaw-skills) — 5400+ OpenClaw 技能合集

---

## 中国用户适配

国内自由职业者/个体户的业务环境与海外有显著差异，尤其在发票和报税环节。以下如实说明各模块的适配现状。

### 可直接复用的模块

| 模块 | 适配方案 | 说明 |
|------|---------|------|
| 客户 CRM | 原方案直接可用 | SQLite + 邮件扫描，无地域限制 |
| 时间追踪 | Kimai 技能直接可用 | 开源自托管，无地域限制 |
| 排期管理 | 飞书日历 / 钉钉日历替代 Google Calendar | 需对应的技能或 API 集成 |
| 报价单 | 原方案直接可用 | 纯文本生成，无地域限制 |
| 范围蔓延检测 | 原方案直接可用 | 基于本地 SOW 比对 |

### 推送渠道替代

| 原版方案 | 国内替代 | 集成方式 |
|---------|---------|---------|
| Telegram | **钉钉群机器人** | Webhook，最简单 |
| Telegram | **飞书机器人** | 支持富文本卡片 |
| Telegram | **企业微信应用** | 企业用户首选 |

### 发票模块：现状与限制

**核心差异**：国内使用增值税发票体系（金税系统），与海外的 FreshBooks/Stripe 发票体系完全不同。

| 维度 | 海外 | 国内 |
|------|------|------|
| 发票生成 | FreshBooks/Stripe，API 完善 | 需通过税控设备（如百望云、航信诺诺）开具 |
| 发票类型 | 商业发票（Invoice），格式自由 | 增值税普通发票/专用发票，格式固定 |
| 催款 | 邮件即可 | 邮件 + 微信，文化上更依赖关系 |

**当前可行方案**：

1. **记账与费用分类**：使用「自记账」（zijizhang.com）等小微企业记账工具，或用 OpenClaw 直接管理本地 SQLite 分类账
2. **发票查验**：GitHub 上有 `openclaw-invoice-plugin-mvp`（百望发票查验插件），可验证收到的发票真伪
3. **开票提醒**：OpenClaw 可以追踪哪些项目需要开票，但实际开票仍需登录税控系统手动操作

**暂不可行的功能**：

- 自动开具增值税发票（金税系统无公开 API）
- 自动报税（需通过电子税务局，无标准 API 接入）

> 如果你是个体户或小规模纳税人，建议的务实方案是：让 OpenClaw 负责「记录和提醒」（哪个客户该开票了、本月收入汇总、费用分类），实际开票和报税操作仍由人工完成或委托代账公司。

### 适配提示词

将原版提示词中的海外工具替换为国内方案：

```text
你是我的自由职业业务助手。管理客户、工时、开票提醒和排期。

每日例行任务（定时任务）：
1. 早 8 点：扫描邮件，更新客户互动记录，推送到钉钉"业务助手"群
2. 早 8:30：检查今日日历，准备客户简报
3. 晚 6 点：汇总今日工时，标记未记录的时间段
4. 每周五下午 5 点：生成周报——本周工时、已开票金额、未开票项目、待收款

开票提醒：
- 项目完成里程碑时，提醒我登录税控系统开票
- 维护一份"待开票清单"，包含客户名、金额、项目名
- 开票后我手动标记为"已开"，你更新记录

催款（适配国内习惯）：
- 逾期 3 天：微信友好提醒（帮我草拟消息）
- 逾期 7 天：邮件正式催款
- 逾期 15 天：电话催款提醒 + 草拟正式催款函

费用分类：
- 按以下类别归档支出：办公设备、软件订阅、交通差旅、外包费用、其他
- 每月底生成费用汇总，方便季度申报或交给代账公司
```

### 相关链接（国内）

- [自记账](https://www.zijizhang.com/) — 小微企业/个体户自助记账报税
- [百望发票查验插件](https://github.com/fenghaozhe-git/openclaw-invoice-plugin-mvp) — OpenClaw 发票验真
- [OpenClaw 中文社区](https://clawd.org.cn/) — 飞书/钉钉/企业微信集成文档
- [AKShare](https://github.com/akfamily/akshare) — 如需追踪投资收入，可配合财报追踪用例

---

**原文链接**：本用例为中文原创，参考了 [OpenClaw Freelancer Command Center](https://popularaitools.ai/openclaw-freelancer-review/) 和 [MindStudio Freelancer Agents](https://www.mindstudio.ai/blog/ai-agents-for-freelancers) 等多个来源。
