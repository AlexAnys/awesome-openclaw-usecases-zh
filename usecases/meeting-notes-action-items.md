# 会议纪要与待办事项自动化

> 翻译自 [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)，含国内适配（飞书妙记 / 腾讯会议 / 钉钉）。

刚开完一场 45 分钟的团队会议，你需要写会议纪要、提取行动项，然后分别录入 Jira、Linear 或待办清单——全靠手动。等你处理完，下一场会议已经开始了。如果在转录文本落地的那一刻，你的智能体就自动搞定这一切呢？

这个用例把任何会议转录文本变成结构化纪要，并自动在项目管理工具中创建对应的任务。

## 痛点

会议纪要枯燥却重要。大多数人要么跳过不写（然后丢失上下文），要么花 20 多分钟手写。行动项经常被遗忘——因为它们只存在某人脑子里，或者埋在聊天记录的某个角落。这个智能体消除了"我们讨论过了"和"已记录、已分配"之间的断层。

## 它能做什么

| 功能 | 说明 |
|------|------|
| **监听转录来源** | 支持 Otter.ai 导出、Google Meet 转录、Zoom 录制摘要，或直接粘贴到对话中 |
| **提取关键信息** | 识别决策事项、讨论话题、行动项及其负责人和截止日期 |
| **自动创建任务** | 在 Jira、Linear、Todoist 或 Notion 中创建任务，分配给正确的人，附带会议上下文 |
| **发布纪要摘要** | 将摘要推送到 Slack 或 Discord，让整个团队留存记录 |
| **跟踪提醒** | 可选：在截止日期前通过定时提醒催办负责人 |

## 所需技能

- 项目管理平台集成：Jira、Linear、Todoist 或 Notion（用于任务创建）
- 团队沟通工具集成：Slack 或 Discord（用于发布摘要）
- 文件系统访问（用于读取转录文件）
- 定时任务 / cron（用于跟踪提醒）
- 可选：Otter.ai、Fireflies.ai 或 Google Meet API（用于自动获取转录文本）

## 如何设置

### 1. 选择转录来源

最简单的方式是直接把转录文本粘贴到对话中。如果需要自动化，可以设置文件夹监控或 API 集成。

### 2. 基础用法：粘贴转录并生成纪要

```text
I just finished a meeting. Here's the transcript:

[paste transcript or point to file]

Please:
1. Write a concise summary (max 5 bullet points) covering key decisions and topics.
2. Extract ALL action items. For each one, identify:
   - What needs to be done
   - Who is responsible (match names to my team)
   - Deadline (if mentioned, otherwise mark as "TBD")
3. Create a Jira ticket for each action item, assigned to the right person.
4. Post the full summary to #meeting-notes in Slack.
```

### 3. 进阶：文件夹自动监控

设置一个自动化管道，定期扫描转录文件并处理：

```text
Set up a recurring task: every 30 minutes, check ~/meeting-transcripts/ for
new .txt or .vtt files. When you find one:

1. Parse the transcript into a structured summary with action items.
2. Create tasks in Linear for each action item.
3. Post the summary to #team-updates in Slack.
4. Move the processed file to ~/meeting-transcripts/processed/.

For each action item with a deadline, set a reminder to ping the assignee
in Slack one day before it's due.
```

### 4. 自定义输出格式

```text
When writing meeting summaries, always use this structure:
- **Date & Attendees** at the top
- **Key Decisions** — numbered list of what was decided
- **Action Items** — table with columns: Task, Owner, Deadline, Status
- **Open Questions** — anything unresolved that needs follow-up
```

## 关键洞察

- 真正的价值不在于纪要本身，而在于**自动创建任务**。不转化为可追踪任务的会议纪要只是形式主义
- 可以搭配 [Todoist 任务管理器](todoist-task-manager.md) 用例，获得对智能体创建的任务的完整可视化
- Zoom 或 Google Meet 的 VTT/SRT 字幕文件是很好的输入——它们包含时间戳，帮助智能体将发言归属到具体的人
- 先从最简单的开始（粘贴转录、获取摘要），验证输出质量后再逐步自动化。不要在验证效果之前就过度设计管道

## 中国用户适配

国内企业的会议场景和工具生态与海外有很大不同。以下是针对国内环境的适配方案。

### 国内会议痛点

据统计，国内企业因会议效率低下造成的损失平均占年度营收的 11%，其中会议纪要处理不当是主要诱因之一。常见痛点包括：

- **会议太多**：员工被迫参加大量不必要的会议，纪要负担沉重
- **纪要与执行脱节**：纪要写了没人看，行动项分散在聊天群里无人跟进
- **转写工具碎片化**：飞书妙记、腾讯会议智能纪要、钉钉 AI 听记各有各的封闭生态，纪要数据难以跨平台流转

### 转录来源替代方案

| 原版工具 | 国内替代 | 说明 |
|---------|---------|------|
| Otter.ai / Fireflies.ai | **飞书妙记** | 支持 19 种语言，准确率 98%，可导出飞书文档/TXT/SRT 格式 |
| Google Meet 转录 | **腾讯会议智能纪要** | 提供 REST API 获取转写段落和纪要摘要（需商业版/企业版） |
| Zoom 录制摘要 | **钉钉 AI 听记** | 支持 72 种语言转写，可一键同步待办到钉钉待办 |
| 直接粘贴 | **任何工具导出后粘贴** | 最通用的方式，不依赖特定平台 |

### 任务创建替代方案

| 原版工具 | 国内替代 | OpenClaw 技能 |
|---------|---------|---------------|
| Jira / Linear | **飞书项目（Meego）** | 通过 [feishu-doc](https://playbooks.com/skills/openclaw/openclaw/feishu-doc) 写入飞书文档，或使用飞书项目 API |
| Todoist | **滴答清单** | [ticktick-api-skill](https://playbooks.com/skills/openclaw/skills/ticktick-api-skill) |
| Notion | **飞书文档 / 钉钉文档** | [feishu-doc](https://playbooks.com/skills/openclaw/openclaw/feishu-doc) |
| Slack / Discord | **飞书群聊 / 钉钉群聊** | 通过飞书/钉钉渠道插件直接发送 |

### 飞书用户方案（推荐）

飞书生态内可以实现最完整的闭环：会议录制 -> 妙记转写 -> OpenClaw 处理 -> 飞书文档 + 任务。

**前置条件**：已完成 [飞书 AI 助手](cn-feishu-ai-assistant.md) 的基础接入。

**安装飞书相关技能**：

```bash
npx playbooks add skill openclaw/openclaw --skill feishu-doc
npx playbooks add skill openclaw/skills --skill feishu-calendar-tool
```

**使用方式**：会议结束后，从飞书妙记导出转写文本（TXT 或飞书文档格式），然后在飞书中发送给 OpenClaw 机器人：

```text
这是今天下午产品评审会的转录文本：
[粘贴妙记导出的文本]

请：
1. 整理成结构化会议纪要（决策事项 + 行动项 + 待确认事项）
2. 把纪要写入飞书文档，放到"会议纪要"文件夹
3. 每个行动项标注负责人和截止日期
4. 把纪要摘要发到产品组群聊
```

### 腾讯会议用户方案

腾讯会议开放平台提供了录制转写 API（商业版/企业版），可通过 API 自动获取会议纪要：

- 查询录制转写段落信息：`GET /v1/records/{record_file_id}/transcripts`
- 查询智能纪要：`GET /v1/smart/minutes/{record_file_id}`

获取到转写文本后，可通过 OpenClaw 进一步结构化处理并分发到对应的任务管理工具。

### 钉钉用户方案

钉钉 AI 听记支持将会议录音自动转写，并可一键同步待办到钉钉待办。2025 年底升级后支持图文纪要和 72 种语言。

**前置条件**：已完成 [钉钉 AI 助手](cn-dingtalk-ai-assistant.md) 的基础接入。

**使用方式**：会议结束后，从钉钉 AI 听记导出纪要文本，发送给 OpenClaw 钉钉机器人进行进一步处理和任务分发。

### 实用建议

- **先用最简单的方式验证**：不管用什么会议工具，先手动复制转写文本粘贴给 OpenClaw，确认输出质量满意后再考虑自动化
- **飞书妙记 + OpenClaw 是目前最顺畅的组合**：飞书妙记导出格式规范，OpenClaw 飞书技能生态最成熟
- **跨平台场景**：如果团队同时使用多个会议工具，统一导出为 TXT 后交给 OpenClaw 处理，用同一套模板输出
- **安全提醒**：会议转录可能包含敏感商业信息，请确保 OpenClaw 运行在安全的环境中，不要将转录文本发送到不受信任的第三方服务

> **安全提示**：腾讯会议 API 凭证和飞书应用密钥属于敏感信息，请通过环境变量或 `.env` 文件配置，确保 `.env` 已加入 `.gitignore`。

## 相关链接

- [Otter.ai](https://otter.ai/)
- [Jira REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [Linear API](https://developers.linear.app/)
- [Slack API](https://api.slack.com/)
- [飞书妙记](https://www.feishu.cn/product/minutes) — 飞书官方会议转写工具
- [腾讯会议开放平台](https://meeting.tencent.com/open-api.html) — 腾讯会议 API 文档
- [钉钉 AI 听记](https://www.dingtalk.com/markets/dingtalk-ai) — 钉钉会议转写功能
- [feishu-doc 技能 - ClawHub](https://playbooks.com/skills/openclaw/openclaw/feishu-doc)
- [ticktick-api-skill 技能 - ClawHub](https://playbooks.com/skills/openclaw/skills/ticktick-api-skill)
- [feishu-calendar-tool 技能 - ClawHub](https://playbooks.com/skills/openclaw/skills/feishu-calendar-tool)
