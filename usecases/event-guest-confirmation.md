# 活动嘉宾确认

你正在举办一个活动——晚宴派对、婚礼、公司团建——你需要确认一份嘉宾名单上的出席情况。手动打电话给 20 多个人非常繁琐：你要反复打电话、忘记谁说了什么、还会弄丢饮食限制或携伴信息。发短信有时管用，但人们会忽略消息。真正的电话通话能获得更高的回复率。

这个用例让 OpenClaw 使用 [SuperCall](https://clawhub.ai/xonder/supercall) 插件逐一给你名单上的每位嘉宾打电话，确认他们是否出席，收集备注信息，并将所有结果汇总成一份摘要。

## 功能介绍

- 遍历嘉宾名单（姓名 + 电话号码）并逐一拨打
- AI 以友好的角色介绍自己是你的活动协调员
- 向嘉宾确认活动日期、时间和地点
- 询问是否出席，并收集备注信息（饮食需求、携伴、到达时间等）
- 所有电话完成后，汇总摘要：谁确认了、谁拒绝了、谁没接电话，以及所有备注

## 为什么选择 SuperCall

这个用例专门使用 [SuperCall](https://clawhub.ai/xonder/supercall) 插件——而不是内置的 `voice_call` 插件。关键区别在于：SuperCall 是一个完全独立的语音智能体。通话中的 AI 角色**只能访问你提供的上下文**（角色名称、目标和开场白）。它无法访问你的网关智能体、你的文件、你的其他工具或任何其他内容。

这对嘉宾确认很重要，因为：

- **安全性**：电话另一端的人无法通过对话操纵或访问你的智能体。没有提示注入（prompt injection）或数据泄露的风险。
- **更好的对话**：因为 AI 被限定在单一聚焦任务（确认出席），它能保持话题并比通用语音智能体更自然地处理通话。
- **适合批量处理**：你要给不同的人打很多电话。一个每次通话都重置的沙箱化角色正是你所需要的——对话之间不会互相影响。

## 所需技能

- [SuperCall](https://clawhub.ai/xonder/supercall) — 通过 `openclaw plugins install @xonder/supercall` 安装
- 一个带电话号码的 Twilio 账户（用于拨打外呼电话）
- 一个 OpenAI API 密钥（用于 GPT-4o 实时语音 AI）
- ngrok（用于 webhook（网络钩子）隧道——免费版即可）

完整配置说明请参阅 [SuperCall README](https://github.com/xonder/supercall)。

## 设置方法

1. 按照[设置指南](https://github.com/xonder/supercall#configuration)安装并配置 SuperCall。确保在你的 OpenClaw 配置中启用了 hooks。

2. 准备你的嘉宾名单。你可以直接在聊天中粘贴，也可以保存在文件中：

```text
Guest List — Summer BBQ, Saturday June 14th, 4 PM, 23 Oak Street

- Sarah Johnson: +15551234567
- Mike Chen: +15559876543
- Rachel Torres: +15555551234
- David Kim: +15558887777
```

3. 给 OpenClaw 设置提示词：

以下提示词让智能体逐一拨打嘉宾电话并汇总确认结果：

```text
I need you to confirm attendance for my event. Here are the details:

Event: Summer BBQ
Date: Saturday, June 14th at 4 PM
Location: 23 Oak Street

Here is my guest list:
<paste your guest list here>

For each guest, use supercall to call them. Use the persona "Jamie, event coordinator
for [your name]". The goal for each call is to confirm whether they're attending,
and note any dietary restrictions, plus-ones, or other comments.

After each call, log the result. Once all calls are done, give me a full summary:
- Who confirmed
- Who declined
- Who didn't answer
- Any notes or special requests from each guest
```

4. OpenClaw 会使用 SuperCall 逐一拨打每位嘉宾的电话，然后汇总结果。你可以随时询问进度更新。

## 关键洞察

- **从小规模测试开始**：先用 2-3 位嘉宾试试，确保角色和开场白听起来合适。你可以在拨打完整名单之前调整语调。
- **注意拨打时间**：不要安排太早或太晚的电话。你可以告诉 OpenClaw 只在特定时间段内拨打。
- **查看通话记录**：SuperCall 会将通话记录保存到 `~/clawd/supercall-logs`。在第一批电话后浏览一下，看看对话进行得如何。
- **未接电话处理**：如果有人没接电话，OpenClaw 会记录下来，你可以决定是稍后重试还是改用短信跟进。
- **真实电话通话需要花钱**：每次通话都会消耗 Twilio 通话时长。在你的 Twilio 账户中设置适当的限额，尤其是嘉宾名单较大时。

## 相关链接

- [SuperCall on ClawHub](https://clawhub.ai/xonder/supercall)
- [SuperCall on GitHub](https://github.com/xonder/supercall)
- [Twilio Console](https://console.twilio.com)
- [OpenAI Realtime API](https://platform.openai.com/docs/guides/realtime)
- [ngrok](https://ngrok.com)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/event-guest-confirmation.md)
