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

## 中国用户适配

如果你使用飞书或钉钉，只需要把推送通道和新闻源替换一下，其他逻辑完全一样。

### 推送通道替换

把提示词中的 Telegram / Discord 替换为你实际使用的 IM：

```text
I want to set up a regular morning brief. Every morning at 8:00 AM,
send me a report through Feishu.
```

前提是你已经配好了对应的 IM 通道（参考 [飞书 AI 助手](cn-feishu-ai-assistant.md) 或 [钉钉 AI 助手](cn-dingtalk-ai-assistant.md)）。

### 中文新闻源推荐

把英文新闻源换成中文源，让简报更贴合国内信息环境：

```text
我想设置一个每日早间简报。每天早上 8:00 通过飞书发给我。

内容包括：
1. 与我相关的隔夜新闻（关注 AI、创业、科技方向，优先看 36kr、少数派、知乎热榜）
2. 今天需要完成的任务（从我的待办清单中拉取）
3. 你建议今天可以帮我自动完成的事情

对于内容创意部分，直接写完整草稿，不要只列标题。
```

> **提示**：抓取 36kr、少数派等中文网站需要 OpenClaw 具备网页浏览或搜索能力。如果你的 agent 无法访问这些网站，请确认已安装浏览器技能或网页搜索技能。

### 定时任务（cron）设置

OpenClaw 支持用自然语言创建定时任务：

```text
帮我创建一个定时任务：每个工作日早上 7:30（比原版 8:00 提前，适配国内上班时间），生成早间简报并发到飞书。
时区设为 Asia/Shanghai。
```

也可以用命令行精确控制：

```bash
openclaw cron add \
  --name "morning-brief-cn" \
  --cron "30 7 * * 1-5" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "生成今日早间简报并发送到飞书" \
  --announce \
  --channel feishu
```

## 参考来源

灵感来自 [Alex Finn 关于改变生活的 OpenClaw 用例的视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)。
