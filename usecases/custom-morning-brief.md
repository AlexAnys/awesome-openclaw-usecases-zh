# 自定义早间简报

你每天醒来后的前 30 分钟都在追赶信息——刷新闻、查日历、检查待办事项、试图弄清今天什么最重要。如果这一切都已经完成，以一条短信的形式等着你呢？

这个工作流让 OpenClaw 每天在预定时间向你发送一份完全定制的早间简报，涵盖新闻、任务、创意和主动建议。

## 功能介绍

- 每天在固定时间（例如上午 8:00）向 Telegram、Discord 或 iMessage 发送结构化的早间报告
- 通过浏览网页研究与你兴趣相关的隔夜新闻
- 审查你的待办清单并呈现当天的任务
- 在你睡觉时生成创意产出（完整的脚本、邮件草稿、商业提案——不仅仅是想法）
- 推荐 AI 当天可以自主完成的任务来帮助你

## 痛点

你把最高效的早晨时间花在了解情况上。与此同时，你的 AI 智能体整晚都在闲置。早间简报将闲置的夜间时间变成高效的准备时间——你醒来时工作已经完成。

## 所需技能

- Telegram、Discord 或 iMessage 集成
- Todoist / Apple Reminders / Asana 集成（你使用的任务管理工具）
- [x-research-v2](https://clawhub.ai) 用于社交媒体趋势研究（可选）

## 设置方法

1. 将 OpenClaw 连接到你的消息平台和任务管理器。

2. 给 OpenClaw 设置提示词：

以下提示词让智能体每天早上自动生成并发送定制的早间简报：

```text
I want to set up a regular morning brief. Every morning at 8:00 AM,
send me a report through Telegram.

I want this report to include:
1. News stories relevant to my interests (AI, startups, tech)
2. Ideas for content I can create today
3. Tasks I need to complete today (pull from my to-do list)
4. Recommendations for tasks you can complete for me today

For the content ideas, write full draft scripts/outlines — not just titles.
```

3. OpenClaw 会自动安排定时任务。第二天早上检查你的消息来验证它是否正常工作。

4. 随时间自定义——只需给你的机器人发消息：

以下是一些自定义简报内容的示例指令：

```text
Add weather forecast to my morning brief.
Stop including general news, focus only on AI.
Include a motivational quote each morning.
```

5. 如果你想不出要包含什么内容，不必勉强——只需说：

以下提示词让智能体自主决定简报中最有用的内容：

```text
I want this report to include things relevant to me.
Think of what would be most helpful to put in this report.
```

## 关键洞察

- AI 推荐任务部分是最强大的功能——它让智能体主动思考如何帮助你，而不是等待指令。
- 你可以通过发消息来定制简报。说"在我的早间简报中加入股票价格"，它就会更新。
- 完整草稿（而不仅仅是想法）是节省时间的关键。醒来就看到脚本，而不是建议。
- 无论你从事什么行业——包含任务、新闻和主动建议的早间简报都是普遍有用的。

## 参考来源

灵感来自 [Alex Finn 关于改变生活的 OpenClaw 用例的视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/custom-morning-brief.md)
