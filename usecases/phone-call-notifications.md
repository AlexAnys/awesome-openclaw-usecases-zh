# 电话通知提醒

推送通知太容易被忽略——淹没在几十条未读消息里，关键提醒和普通聊天看起来一模一样。当真正紧急的事情发生时，你需要一种"打断"级别的通知方式。

这个用例让 OpenClaw 在判定情况紧急时直接给你打电话。不是推送通知，是真正的电话铃声。你接起来，听到提醒内容，还可以实时追问细节。

## 功能介绍

- 智能体监控各类数据源（邮件、股价、日历、服务器状态等），根据预设的紧急阈值判断是否需要电话通知
- 通过 [clawr.ing](https://clawr.ing) 发起真实电话呼叫——无需配置 Twilio 账号、无需申请电话号码、无需搭建 webhook
- 接通后支持双向对话：你可以追问细节，智能体实时回答
- 搭配定时任务或心跳检查使用，实现全自动监控 + 电话告警

## 痛点

你设置了十几个推送通知渠道，但真正紧急的事情发生时，你正在开会、开车或睡觉——推送通知被划走了，或者根本没注意到。电话呼叫是唯一能真正"打断"你的通知方式。

## 所需技能

- [clawr.ing](https://clawr.ing) —— 托管式电话呼叫服务，粘贴一段设置提示词即可启用，无需额外配置

可选搭配（取决于你要监控什么）：

- 邮件集成（Gmail / Outlook）
- 日历集成（Google Calendar / Outlook）
- 股票/行情数据技能
- 网页浏览或搜索技能
- 服务器监控工具

## 如何设置

1. 前往 [clawr.ing 控制台](https://clawr.ing) 注册账号，选择一个语音风格。

2. 将控制台提供的设置提示词粘贴到 OpenClaw 对话中。完成后智能体就具备了打电话的能力。

3. 告诉智能体什么时候该打电话给你：

```text
I want you to call me only when something is truly urgent. Here are my rules:

- Call me if I get an email from my boss or any C-level exec
- Call me if any stock in my watchlist moves more than 5% in either direction
- Call me if my server monitoring detects downtime
- For everything else, just send me a chat message

When you call, start with a one-sentence summary of why you're calling, then let me ask questions.
```

4. 搭配定时任务实现周期性监控：

```text
Every 30 minutes during market hours (9:30 AM – 3:00 PM), check my stock watchlist.
If any position moves more than 5%, call me immediately.
Otherwise, send a summary to chat at end of day.
```

## 使用场景示例

### 早间语音简报

```text
Call me every morning at 7:30 AM with a briefing. Keep it under 2 minutes.
Include: weather, today's calendar, any urgent emails overnight, and top 3 news stories relevant to my interests.
```

### 股价异动告警

```text
Monitor these stocks: AAPL, TSLA, NVDA.
If any of them drops more than 3% within an hour, call me immediately.
When you call, tell me which stock, the percentage change, and any news that might explain the move.
```

### 重要邮件实时通知

```text
During business hours (9 AM – 6 PM), check my email every 5 minutes.
If I receive an email from anyone at @importantclient.com, call me right away.
Read me the subject line and first two sentences.
```

## 使用建议

- **控制呼叫频率**：如果你的智能体一天打 10 个电话，你很快就会开始忽略它。仔细区分什么情况值得打电话、什么用聊天消息就够了
- **用快速模型**：电话对话的延迟很敏感，建议在语音交互场景使用响应速度快的模型
- **搭配心跳/定时任务**：让智能体周期性检查数据源，而不是被动等待事件触发
- **善用双向对话**：接到电话后可以直接追问"细节是什么？""帮我查一下最新消息"，智能体会实时回答

## 隐私说明

clawr.ing 在传输过程中加密音频数据，通话结束后不保留录音或文字记录。

---

## 中国用户适配

### 核心问题：clawr.ing 在国内的可用性

clawr.ing 声称覆盖 100+ 个国家，包括"大部分亚洲地区"，但 **未明确列出中国大陆为支持地区**。即使技术上可以接通，国内用户使用海外电话通知服务还面临以下问题：

- **来电显示为陌生海外号码**，极大概率被手机系统或运营商标记为骚扰/诈骗电话并自动拦截
- **通话质量和延迟不可控**，跨国语音链路不稳定
- **费用较高**，国际通话按分钟计费，东亚地区费率高于欧美
- **合规风险**：国内对自动外呼有严格监管（需呼叫中心许可证、号码备案、通话录音留存等），使用海外服务绕过监管可能存在法律风险

**结论：直接使用 clawr.ing 在国内不具备实际可操作性。**

### 推荐替代方案：IM 机器人推送

对于大多数国内用户，**飞书/钉钉/企业微信机器人推送**是比电话通知更实用、更可靠的"紧急通知"方案：

| 方案 | 优势 | 局限 |
|------|------|------|
| **飞书机器人** | 支持富文本/卡片消息、可 @指定人、支持紧急标记 | 需要飞书组织账号 |
| **钉钉机器人** | 支持 DING 消息（电话/短信级提醒）、普及率高 | Webhook 需配置安全策略 |
| **企业微信机器人** | 与微信生态打通、支持模板消息 | 需要企业认证 |

其中**钉钉的 DING 消息**最接近电话通知的效果——它会以电话或短信形式提醒对方，且在国内合规框架内。

配置方式参考：[钉钉 AI 助手](cn-dingtalk-ai-assistant.md) | [飞书 AI 助手](cn-feishu-ai-assistant.md) | [企业微信 AI 助手](cn-wecom-ai-assistant.md)

### 进阶方案：国内语音通知服务

如果你确实需要电话级别的通知（例如服务器宕机告警、交易风控），可以考虑接入国内云通信平台：

- **[阿里云语音通知](https://www.aliyun.com/product/vms)**：支持语音通知和语音验证码，按次计费，需企业实名认证 + 语音模板审核
- **[腾讯云语音消息](https://cloud.tencent.com/product/vms)**：类似阿里云方案，与腾讯生态集成
- **[容联云](https://www.yuntongxun.com/)**：提供语音通知 API，支持 TTS 和录音播放

**注意事项**：
- 国内语音通知服务**均需企业实名认证**，个人用户无法直接使用
- 需要提前**报备号码和语音模板**，不支持任意内容的动态播报
- 自动外呼受工信部严格监管，投诉率超标会被停机
- 这些服务目前没有现成的 OpenClaw 技能，需要自行通过 MCP 服务器或自定义技能对接

### 实用建议

对于个人用户，推荐的方案优先级：

1. **钉钉 DING 消息**（最接近电话效果，合规，免费）
2. **飞书/企业微信紧急消息**（富文本 + @提醒，信息量更大）
3. **多通道组合**：紧急事项同时推送到 IM + 短信（通过阿里云短信服务），确保触达
4. **阿里云/腾讯云语音通知**（仅适合企业用户，配置门槛高）

> **提示**：如果你的核心需求是"不错过紧急信息"，与其追求电话通知，不如在 IM 端做好分级——用 DING 消息或紧急标记确保关键信息的触达率，效果更好、成本更低。

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/phone-call-notifications.md)
