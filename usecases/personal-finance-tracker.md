# 个人记账与财务管理

> 含国内适配：飞书语音记账 / 支付宝微信账单分析

你想搞清楚每个月钱花到哪里去了，但手动记账坚持不了三天，银行 App 的分类又不够智能。

这个工作流把 OpenClaw 变成你的私人财务助手：

- **随手记录**：通过聊天或语音说一句"午餐 35 元"，自动分类入账
- **收据识别**：拍照发给 Telegram/飞书机器人，OCR（光学字符识别）自动提取金额和商家
- **预算监控**：设定月度预算，超支时主动提醒
- **月度报告**：每月自动生成支出分析，按分类对比预算，识别异常消费

## 它能做什么

| 功能 | 说明 |
|------|------|
| 自然语言记账 | 发消息"打车 28"自动归类到"交通" |
| 收据 OCR | 拍照→提取金额/商家/日期→自动入账 |
| 预算追踪 | 设定各分类预算上限，接近时告警 |
| 支出分析 | 按日/周/月维度，分类汇总 + 趋势对比 |
| 账单导入 | 解析银行/支付宝/微信导出的 CSV 账单 |
| 定期报告 | cron job 定时推送周报/月报 |

## 所需技能

- `smart-expense-tracker`（OpenClaw 官方技能，自然语言记账 + 预算追踪）
- `personal-finance`（SQLite 后端，支持 EMI（每月等额还款）提醒和年度统计）
- Telegram 或飞书频道（接收报告和告警）
- OpenClaw 的 cron job（定时任务）支持

可选：

- `actual-budget`（对接 Actual Budget 开源预算软件）
- ezBookkeeping（轻量自托管记账应用，有官方 OpenClaw 技能）

### API / 依赖接入现状

> ⚠️ 诚实说明：银行数据自动同步在全球范围内都有门槛，国内尤其如此。

| 方案 | 可用性 | 说明 |
|------|--------|------|
| **聊天记账**（核心方案） | ✅ 立即可用 | 无需任何外部 API，数据存本地 SQLite |
| **收据 OCR** | ✅ 立即可用 | OpenClaw 内置视觉能力，Telegram/飞书发图即可 |
| **CSV 账单导入** | ✅ 可用 | 支付宝/微信/银行均可导出 CSV，手动操作 |
| **Plaid 银行同步**（国际） | ⚠️ 有门槛 | 需注册 Plaid 开发者账号，支持美/加/英/欧银行 |
| **支付宝/微信 API** | ❌ 个人不可用 | 不对个人开发者开放账单读取 API |

**本用例聚焦前三种方案**——不依赖银行 API，读者照做即可复现。

## 如何设置

### 方案一：聊天记账 + 自动分析 ⭐

最简方案，5 分钟上手。

1. 安装 `smart-expense-tracker` 技能：

```bash
npx playbooks add skill openclaw/skills --skill smart-expense-tracker
```

2. 向 OpenClaw 发送设置提示词：

```text
I want you to be my personal expense tracker. Here are the rules:

1. When I send a message like "lunch 35" or "coffee 18 Starbucks", log it as an expense with:
   - Amount
   - Category (auto-detect from context: Food, Transport, Shopping, Entertainment, Health, Utilities, Misc)
   - Merchant (if mentioned)
   - Date (today unless specified)

2. When I say "report" or "月报", generate a spending summary:
   - Total spend vs budget
   - Breakdown by category (with bar chart using text)
   - Top 3 categories
   - Comparison with last month if data exists

3. My monthly budget is ¥8000. Alert me when any category exceeds 80% of its budget.

4. Budget allocation:
   - Food: ¥2500
   - Transport: ¥800
   - Shopping: ¥1500
   - Entertainment: ¥500
   - Utilities: ¥1200
   - Health: ¥500
   - Misc: ¥1000

Confirm you understand, then I'll start logging.
```

日常使用示例：

```text
午餐 35 公司食堂
打车 28.5
买书 89 京东
电费 156
```

OpenClaw 会自动将每条消息解析为结构化记录并存入本地数据库。

### 方案二：收据 OCR + 聊天记账 ⭐⭐

在方案一的基础上，增加拍照记账。

1. 确保 OpenClaw 连接了 Telegram 或飞书（参考各平台接入指南）。

2. 向 OpenClaw 补充提示词：

```text
In addition to text logging, when I send a photo of a receipt:
1. Use OCR to extract: merchant name, total amount, date, and line items
2. Log the total as an expense, auto-categorize based on merchant
3. Reply with the extracted details so I can verify
4. If OCR is uncertain about any field, ask me to confirm

For Chinese receipts, parse amount fields like "合计" or "实收" as the total.
```

3. 使用方式：直接在 Telegram/飞书对话中拍照发送收据，OpenClaw 自动识别并入账。

### 方案三：CSV 账单导入 + 深度分析 ⭐⭐

适合月末集中分析。

1. 导出账单：
   - **支付宝**：我的 → 账单 → 右上角 `···` → 资金明细 → 用于个人对账 → 申请导出（发送至邮箱，Excel 格式）
   - **微信支付**：我 → 服务 → 钱包 → 账单 → 右上角 `···` → 导出账单
   - **银行 App**：各家银行通常支持导出 CSV/PDF 交易记录

2. 将导出的文件发送给 OpenClaw，附上分析指令：

```text
I'm sending you my Alipay/WeChat payment export for last month. Please:

1. Parse all transactions from the CSV/Excel file
2. Categorize each transaction (Food, Transport, Shopping, Entertainment, Utilities, Subscriptions, Transfer, Other)
3. Generate a report with:
   - Total income vs total expenses
   - Expense breakdown by category (table + text bar chart)
   - Top 10 merchants by spend
   - Recurring charges (subscriptions, memberships)
   - Unusual transactions (significantly above average for their category)
4. Compare with my budget: [paste your budget here]
5. Suggest 3 actionable ways to reduce spending next month

Format the report in Chinese.
```

### 方案四：自托管记账系统对接 ⭐⭐⭐

适合偏好完整记账应用的用户。

1. 部署 ezBookkeeping（[GitHub](https://github.com/mayswind/ezbookkeeping)，Go + Vue，MIT 许可）：

```bash
docker run -d -p 8080:8080 mayswind/ezbookkeeping
```

2. 在 ezBookkeeping 中创建 API Token。

3. 配置环境变量：

```bash
# 在 OpenClaw 的 .env 文件中添加
EBKTOOL_SERVER_BASEURL=http://localhost:8080
EBKTOOL_TOKEN=your_api_token_here
```

4. 安装 ezBookkeeping 的 OpenClaw 技能（参考 [官方集成文档](https://ezbookkeeping.mayswind.net/agent/openclaw)）。

5. 此后通过聊天记录的支出会自动同步到 ezBookkeeping，你可以在 Web 界面查看图表和详细报表。

## 实用建议

- **从方案一开始**：先用最简方案坚持一周，确认习惯养成后再升级
- **固定记账时机**：每次消费后立即发一条消息，比晚上回忆更准确
- **每周快速回顾**：设一个周日的 cron job 推送周报，花 2 分钟扫一眼
- **预算要留余量**：初始预算建议比实际目标宽松 10-20%，避免频繁告警导致"通知疲劳"
- **隐私优先**：所有方案的数据都存储在本地（SQLite 或自托管应用），不经过第三方云端

## 相关链接

- [smart-expense-tracker 技能](https://playbooks.com/skills/openclaw/skills/smart-expense-tracker) — 官方自然语言记账技能
- [personal-finance 技能](https://playbooks.com/skills/openclaw/skills/personal-finance) — SQLite 后端财务管理技能
- [intelligent-budget-tracker 技能](https://playbooks.com/skills/openclaw/skills/intelligent-budget-tracker) — AI 驱动的预算追踪技能
- [ezBookkeeping](https://github.com/mayswind/ezbookkeeping) — 开源轻量自托管记账应用，支持 OpenClaw 集成
- [Wally / WallyGPT](https://wally.me/) — 零界面 AI 财务助手，对话式管理个人财务
- [Beam.ai Budget Agent](https://beam.ai/agents/budget-management-agent/) — AI 预算管理 Agent 模板参考

---

## 中国用户适配

### 支付数据获取

国内个人开发者无法通过 API 直接读取支付宝/微信的交易数据。以下是可行的替代方案：

| 方案 | 操作方式 | 适合场景 |
|------|---------|---------|
| **手动导出 CSV** | 支付宝/微信客户端导出账单 → 发送给 OpenClaw 分析 | 月度复盘 |
| **聊天即时记录** | 每笔消费发消息给 OpenClaw | 日常记账 |
| **收据拍照** | 发票/小票拍照发送 | 线下消费 |
| **邮件账单** | 信用卡月账单邮件转发给 OpenClaw | 信用卡用户 |

### 推送渠道适配

| 原版方案 | 国内替代 | 说明 |
|---------|---------|------|
| Telegram | **飞书机器人** | 支持富文本卡片，适合发送报表 |
| Telegram | **钉钉群机器人** | Webhook 方式，配置最简单 |
| Telegram | **企业微信应用** | 企业用户首选 |

### 飞书语音记账设置

如果你已将 OpenClaw 接入飞书（参考 [飞书接入教程](https://www.feishu.cn/content/article/7613711414611463386)），可以直接通过语音消息记账：

1. 在飞书中向 OpenClaw 机器人发送语音消息（如"午餐三十五元"）
2. OpenClaw 通过 Whisper 等语音识别模型转为文字
3. 自动解析金额和分类，入账并回复确认

### 国内记账工具生态参考

| 工具 | 类型 | 说明 |
|------|------|------|
| **咔皮记账**（商汤科技） | AI 记账 App | 拍照/语音自动记账，核心功能永久免费，500 万+用户 |
| **随手记 / 挖财** | 传统记账 App | 功能成熟但无 OpenClaw 集成 |
| **ezBookkeeping** | 自托管 | 国人开发，Go + Vue，支持 OpenClaw 技能 |

> 💡 如果你只需要移动端记账，咔皮记账等原生 App 可能更方便。OpenClaw 方案的优势在于**高度定制化**（自定义分类规则、跨平台数据整合、自动化报表）和**数据完全自主可控**。

### 提示词适配

将英文提示词替换为中文版本，并加入国内支付场景：

```text
你是我的个人财务助手。规则如下：

1. 当我发送类似"午餐 35"、"打车 28 滴滴"的消息时，记录为一笔支出：
   - 金额（单位：人民币）
   - 分类（自动识别：餐饮、交通、购物、娱乐、居住、医疗、其他）
   - 商家（如有提及）
   - 日期（默认今天）

2. 当我发送支付宝或微信导出的账单文件时：
   - 解析所有交易记录
   - 自动分类
   - 生成分析报告

3. 当我说"报告"或"月报"时，生成支出摘要：
   - 总支出 vs 预算
   - 按分类汇总（表格 + 文字柱状图）
   - 消费最高的 3 个分类
   - 与上月对比

4. 我的月度预算是 ¥8000，分配如下：
   - 餐饮：¥2500
   - 交通：¥800
   - 购物：¥1500
   - 娱乐：¥500
   - 居住：¥1200
   - 医疗：¥500
   - 其他：¥1000

5. 任何分类使用超过 80% 预算时提醒我。

确认理解后，我开始记账。
```

### 合规提醒

- 个人财务数据属于敏感信息，建议使用本地部署方案
- 支付宝/微信账单导出文件可能包含支付密码等敏感字段，分析前确认文件内容
- 本用例仅用于个人财务管理，不涉及支付交易操作
