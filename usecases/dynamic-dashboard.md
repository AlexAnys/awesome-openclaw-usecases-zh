# 动态仪表盘与子智能体生成

静态仪表盘显示的是过时的数据，需要持续手动更新。你需要跨多个数据源的实时可见性，又不想构建自定义前端或触发 API 速率限制。

这个工作流创建一个实时仪表盘，通过生成子智能体（sub-agent）并行获取和处理数据：

- 同时监控多个数据源（API、数据库、GitHub、社交媒体）
- 为每个数据源生成子智能体，避免阻塞并分散 API 负载
- 将结果聚合到统一的仪表盘中（文本、HTML 或 Canvas）
- 每 N 分钟用最新数据更新
- 当指标超过阈值时发送警报
- 在数据库中维护历史趋势以供可视化

## 痛点

构建自定义仪表盘需要数周时间。等完成时，需求已经变了。顺序轮询多个 API 既慢又容易触及速率限制。你现在就需要洞察，而不是花一个周末写代码。

## 功能介绍

你通过对话定义要监控的内容："跟踪 GitHub 星标、Twitter 提及、Polymarket 交易量和系统健康状况。" OpenClaw 生成子智能体并行获取每个数据源，聚合结果，并以格式化的仪表盘发送到 Discord 或生成 HTML 文件。更新按定时任务（cron job）自动运行。

仪表盘示例板块：
- **GitHub**：星标、Fork、未关闭的 Issue、最近的提交
- **社交媒体**：Twitter 提及、Reddit 讨论、Discord 活动
- **市场**：Polymarket 交易量、预测趋势
- **系统健康**：CPU、内存、磁盘使用率、服务状态

## 所需技能

- 子智能体生成用于并行执行
- `github`（gh CLI）用于 GitHub 指标
- `bird`（Twitter）用于社交数据
- `web_search` 或 `web_fetch` 用于外部 API
- `postgres` 用于存储历史指标
- Discord 或 Canvas 用于渲染仪表盘
- 定时任务（cron job）用于定期更新

## 设置方法

1. 设置指标数据库：
```sql
-- 指标表
CREATE TABLE metrics (
  id SERIAL PRIMARY KEY,
  source TEXT, -- 例如 "github"、"twitter"、"polymarket"
  metric_name TEXT,
  metric_value NUMERIC,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

-- 警报表
CREATE TABLE alerts (
  id SERIAL PRIMARY KEY,
  source TEXT,
  condition TEXT,
  threshold NUMERIC,
  last_triggered TIMESTAMPTZ
);
```

2. 创建一个用于仪表盘更新的 Discord 频道（例如 #dashboard）。

3. 给 OpenClaw 设置提示词：

以下提示词让智能体每 15 分钟自动获取多源数据并生成仪表盘：

```text
You are my dynamic dashboard manager. Every 15 minutes, run a cron job to:

1. Spawn sub-agents in parallel to fetch data from:
   - GitHub: stars, forks, open issues, commits (past 24h)
   - Twitter: mentions of "@username", sentiment analysis
   - Polymarket: volume for tracked markets
   - System: CPU, memory, disk usage via shell commands

2. Each sub-agent writes results to the metrics database.

3. Aggregate all results and format a dashboard:

📊 **Dashboard Update** — [timestamp]

**GitHub**
- ⭐ Stars: [count] (+[change])
- 🍴 Forks: [count]
- 🐛 Open Issues: [count]
- 💻 Commits (24h): [count]

**Social Media**
- 🐦 Twitter Mentions: [count]
- 📈 Sentiment: [positive/negative/neutral]

**Markets**
- 📊 Polymarket Volume: $[amount]
- 🔥 Trending: [market names]

**System Health**
- 💻 CPU: [usage]%
- 🧠 Memory: [usage]%
- 💾 Disk: [usage]%

4. Post to Discord #dashboard.

5. Check alert conditions:
   - If GitHub stars change > 50 in 1 hour → ping me
   - If system CPU > 90% → alert
   - If negative sentiment spike on Twitter → notify

Store all metrics in the database for historical analysis.
```

4. 可选：使用 Canvas 渲染包含图表的 HTML 仪表盘。

5. 查询历史数据："显示过去 30 天的 GitHub 星标增长情况。"

## 相关链接

- [使用子智能体进行并行处理](https://docs.openclaw.ai/subagents)
- [仪表盘设计原则](https://www.nngroup.com/articles/dashboard-design/)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/dynamic-dashboard.md)
