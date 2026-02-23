# 个人 CRM（客户关系管理）与自动联系人发现

手动记录你见过的人、见面时间以及讨论内容几乎不可能。重要的跟进事项不知不觉就被遗漏，在重要会议前你也忘记了相关背景。

这个工作流自动构建和维护个人 CRM：

- 每日定时任务（cron job）扫描邮件和日历，发现新联系人和互动记录
- 将联系人存储在结构化数据库中，附带关系上下文
- 自然语言查询："我对 [某人] 了解多少？"、"谁需要跟进？"、"我上次和 [某人] 交流是什么时候？"
- 每日会议准备简报：在当天会议前，通过 CRM 和邮件历史研究外部与会者，并提供简报

## 所需技能

- `gog` CLI（用于 Gmail 和 Google Calendar）
- 自定义 CRM 数据库（SQLite 或类似方案），或使用 [crm-query](https://clawhub.ai) 技能（如果可用）
- Telegram 话题用于 CRM 查询

## 如何设置

1. 创建 CRM 数据库：
```sql
CREATE TABLE contacts (
  id INTEGER PRIMARY KEY,
  name TEXT,
  email TEXT,
  first_seen TEXT,
  last_contact TEXT,
  interaction_count INTEGER,
  notes TEXT
);
```
2. 在 Telegram 中设置一个名为 "personal-crm" 的话题用于查询。
3. 指示 OpenClaw：

以下提示词配置自动联系人发现和每日会议简报：

```text
Run a daily cron job at 6 AM to:
1. Scan my Gmail and Calendar for the past 24 hours
2. Extract new contacts and update existing ones
3. Log interactions (meetings, emails) with timestamps and context

Also, every morning at 7 AM:
1. Check my calendar for today's meetings
2. For each external attendee, search my CRM and email history
3. Deliver a briefing to Telegram with: who they are, when we last spoke, what we discussed, and any follow-up items

When I ask about a contact in the personal-crm topic, search the database and give me everything you know.
```

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/personal-crm.md)
