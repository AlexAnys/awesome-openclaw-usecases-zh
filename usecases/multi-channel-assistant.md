# 多渠道个人助手

在多个应用间切换来管理任务、安排日程、发送消息和追踪工作令人精疲力竭。你需要一个统一的接口来连接所有工具。

这个工作流将所有功能整合到一个 AI 助手中：

- Telegram 作为主要交互界面，通过话题路由（不同话题对应视频创意、CRM、收入、配置等）
- Slack 集成用于团队协作（任务分配、知识库保存、视频创意触发）
- Google Workspace：通过聊天创建日历事件、管理邮件、上传到 Drive
- Todoist 用于快速任务捕获
- Asana 用于项目管理
- 自动提醒：垃圾清运日、每周公司通讯等

## 所需技能

- `gog` CLI（Google Workspace）
- Slack 集成（bot + user tokens）
- Todoist API 或技能
- Asana API 或技能
- 配置了多个话题的 Telegram 频道

## 如何设置

1. 为不同场景设置 Telegram 话题：
   - `config` — 机器人设置和调试
   - `updates` — 状态和通知
   - `video-ideas` — 内容管道
   - `personal-crm` — 联系人管理
   - `earnings` — 财务追踪
   - `knowledge-base` — RAG（检索增强生成）数据导入和查询

2. 通过 OpenClaw 配置连接所有工具：
   - Google OAuth（Gmail、Calendar、Drive）
   - Slack（app + user tokens）
   - Todoist API 令牌
   - Asana API 令牌

3. 指示 OpenClaw：

以下提示词配置多渠道路由和自动提醒：

```text
You are my multi-channel assistant. Route requests based on context:

Telegram topics:
- "config" → system settings, debugging
- "updates" → daily summaries, reminders, calendar
- "video-ideas" → content pipeline and research
- "personal-crm" → contact queries and meeting prep
- "earnings" → financial tracking
- "knowledge-base" → save and search content

When I ask you to:
- "Add [task] to my todo" → use Todoist
- "Create a card for [topic]" → use Asana Video Pipeline project
- "Schedule [event]" → use gog calendar
- "Email [person] about [topic]" → draft email via gog gmail
- "Upload [file] to Drive" → use gog drive

Set up automated reminders:
- Monday 6 PM: "Trash day tomorrow"
- Friday 3 PM: "Time to write the weekly company update"
```

4. 逐一测试每个集成，然后测试跨工作流交互（例如，将 Slack 链接保存到知识库，然后在视频研究卡中使用它）。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/multi-channel-assistant.md)
