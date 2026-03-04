# 习惯追踪与自律教练

> 翻译自 [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)，含国内适配（钉钉 / 飞书 / 企业微信通道）。

你试过各种习惯打卡 App——小日常、Forest、番茄 Todo……它们都能用一周，然后你就不再打开了。问题不在 App，而在于追踪习惯是**被动的**。如果你的智能体主动联系你、问你今天过得怎么样、并根据你是在坚持还是在滑落来调整沟通方式呢？

这个用例把 OpenClaw 变成一个**主动的自律教练**，每天通过 Telegram 或短信向你签到。

## 痛点

习惯 App 依赖你主动打开它。推送通知很容易被忽略。真正能促进行为改变的是**主动问责**——有人（或某个东西）直接问你、庆祝你的进步、在你松懈时推你一把。这个智能体就是这样做的，而且不会有麻烦朋友的尴尬。

## 功能说明

- **定时签到**：通过 Telegram 或短信在你选定的时间发送签到消息（例如早 7 点检查晨练、晚 9 点做日末回顾）
- **习惯追踪**：追踪你定义的任何习惯——运动、阅读、冥想、喝水、编程等
- **连续天数追踪**：记录每个习惯的当前连续天数，并在消息中引用
- **自适应提醒**：根据你的表现调整语气（坚持时鼓励、缺勤时温和督促）
- **周报**：总结你的一周，包含完成率、最长连续天数和行为模式（例如"你通常在周三跳过锻炼"）

## 所需技能

- Telegram 或短信集成（短信可用 Twilio，或使用 Telegram Bot API）
- 定时任务（cron job）用于定时签到
- 文件系统或数据库访问，用于存储习惯数据
- 可选：Google Sheets 集成，用于可视化习惯仪表板

## 如何设置

1. 定义你的习惯和签到时间：

```text
I want you to be my accountability coach. Track these daily habits for me:

1. Morning workout (check in at 7:30 AM)
2. Read for 30 minutes (check in at 8:00 PM)
3. No social media before noon (check in at 12:30 PM)
4. Drink 8 glasses of water (check in at 6:00 PM)

Send me a Telegram message at each check-in time asking if I completed
the habit. Keep track of my streaks in a local file.
```

2. 设置追踪方式和语气风格：

```text
When I confirm a habit, respond with a short encouraging message and
mention my current streak. Example: "Day 12 of morning workouts. Solid."

When I miss a habit, don't guilt-trip me. Just acknowledge it and remind
me why I started. If I miss 3 days in a row, send a longer motivational
message and ask if I want to adjust the goal.

If I don't respond to a check-in within 2 hours, send one follow-up.
Don't spam me after that.
```

3. 添加周报功能：

```text
Every Sunday at 10 AM, send me a weekly summary:
- Completion rate for each habit
- Current streaks
- Best day and worst day
- One pattern you noticed (e.g., "You always skip reading on Fridays")
- One suggestion for next week

Store all data in ~/habits/log.json so I can review history anytime.
```

4. 可选——通过 Google Sheets 生成可视化仪表板：

```text
At the end of each day, update a Google Sheet with today's habit data.
Columns: Date, Workout, Reading, No Social Media, Water, Notes.
Mark completed habits with ✓ and missed with ✗.
```

## 关键洞察

- **自适应语气**是关键区别。静态提醒很容易被忽略，但一条"第 15 天了，别现在断"的消息真的能激励你。
- 追踪的习惯数量控制在 3-5 个。追踪太多会导致签到疲劳，你会开始忽略消息。
- 周度模式分析出奇地有用——你会发现"早上有会的日子我从不运动"这样的规律，然后可以提前安排。
- 可以与[健康与症状追踪器](health-symptom-tracker.md)配合使用，关联习惯与身体感受。

## 中国用户适配

### 消息通道选择

原用例推荐 Telegram 和 Twilio SMS，国内用户可根据实际情况选择：

| 通道 | 优势 | 注意事项 |
|------|------|---------|
| **Telegram**（推荐） | 开放 Bot API，配置简单，OpenClaw 原生支持 | 需要科学上网 |
| **钉钉机器人** | 企业用户友好，Stream 模式无需公网 IP | 参考[钉钉 AI 助手](cn-dingtalk-ai-assistant.md)配置 |
| **飞书机器人** | 功能丰富，支持卡片消息展示习惯数据 | 参考[飞书 AI 助手](cn-feishu-ai-assistant.md)配置 |
| **企业微信** | 可通过微信插件触达个人微信 | 参考[企业微信 AI 助手](cn-wecom-ai-assistant.md)配置 |

> **注意**：不建议使用个人微信协议方案（如 WeChaty 等），封号风险极高。

### 国内习惯追踪生态

国内已有成熟的打卡类应用（小日常、Forest、番茄 Todo、小打卡等），但它们都是**被动工具**——需要你主动打开。OpenClaw 的核心价值在于**主动问责**：它会找你，而不是等你来找它。

可以将 OpenClaw 作为这些应用的补充：用打卡 App 做可视化记录，用 OpenClaw 做主动提醒和激励。

### 适配建议

- 习惯内容可根据国内生活调整，例如：早起打卡、背单词、每日步数、控制刷短视频时间等
- 周报可以结合中国节假日和工作日模式分析（如"调休日你的完成率明显下降"）
- 如果使用钉钉/飞书，可以利用其富文本卡片消息让周报更直观

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/habit-tracker-accountability-coach.md)

## 相关链接

- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Twilio SMS API](https://www.twilio.com/docs/sms)
- [Google Sheets API](https://developers.google.com/sheets/api)
