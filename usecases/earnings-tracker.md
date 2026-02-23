# AI 驱动的财报追踪器

在财报季追踪数十家科技公司意味着需要查看多个信息来源并记住报告日期。你希望紧跟 AI/科技公司的财报动态，而不必手动追踪每一家公司。

这个工作流自动化了财报追踪和推送：

- 每周日预览：扫描即将到来的财报日历，将相关的科技/AI 公司发布到 Telegram
- 你选择关注哪些公司，OpenClaw 为每个财报日期安排一次性的 cron job（定时任务）
- 每份报告发布后，OpenClaw 搜索结果，格式化详细摘要（超预期/不及预期、关键指标、AI 亮点），并推送给你

## 所需技能

- `web_search`（内置）
- OpenClaw 的 cron job（定时任务）支持
- 用于财报更新的 Telegram 话题

## 如何设置

1. 创建一个名为 "earnings" 的 Telegram 话题用于接收更新。
2. 向 OpenClaw 发送以下提示词：
```text
Every Sunday at 6 PM, run a cron job to:
1. Search for the upcoming week's earnings calendar for tech and AI companies
2. Filter for companies I care about (NVDA, MSFT, GOOGL, META, AMZN, TSLA, AMD, etc.)
3. Post the list to my Telegram "earnings" topic
4. Wait for me to confirm which ones I want to track

When I reply with which companies to track:
1. Schedule one-shot cron jobs for each earnings date/time
2. After each report drops, search for earnings results
3. Format a summary including: beat/miss, revenue, EPS, key metrics, AI-related highlights, guidance
4. Post to Telegram "earnings" topic

Keep a memory of which companies I typically track so you can auto-suggest them each week.
```

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/earnings-tracker.md)
