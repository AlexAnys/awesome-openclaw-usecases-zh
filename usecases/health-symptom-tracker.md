# 健康与症状追踪器

识别食物敏感性需要长期持续记录，但这种记录非常繁琐难以维持。你需要提醒来记录，也需要分析来发现模式。

这个工作流自动追踪食物和症状：

- 在专用的 Telegram 话题中发送你的饮食和症状信息，OpenClaw 会自动记录所有内容并附带时间戳
- 每天 3 次提醒（早上、中午、晚上）提示你记录餐食
- 随着时间推移，分析模式以识别潜在的触发因素

## 所需技能

- 定时任务（cron job）用于提醒
- Telegram 话题用于记录
- 文件存储（markdown 日志文件）

## 如何设置

1. 在 Telegram 中创建一个名为 "health-tracker"（或类似名称）的话题。
2. 创建日志文件：`~/clawd/memory/health-log.md`
3. 指示 OpenClaw：

以下提示词配置食物和症状的自动追踪与周度分析：

```text
When I message in the "health-tracker" topic:
1. Parse the message for food items and symptoms
2. Log to ~/clawd/memory/health-log.md with timestamp
3. Confirm what was logged

Set up 3 daily reminders:
- 8 AM: "Log your breakfast"
- 1 PM: "Log your lunch"
- 7 PM: "Log your dinner and any symptoms"

Every Sunday, analyze the past week's log and identify patterns:
- Which foods correlate with symptoms?
- Are there time-of-day patterns?
- Any clear triggers?

Post the analysis to the health-tracker topic.
```

4. 可选：为 OpenClaw 添加一个记忆文件，用于追踪已知的触发因素，并随着模式的出现不断更新。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/health-symptom-tracker.md)
