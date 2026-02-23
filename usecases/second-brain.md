# 第二大脑

你会产生想法、发现有趣的链接、听说要读的书——但你从来没有一个好的系统来捕获它们。Notion 变得复杂，Apple Notes 变成了一个拥有 10,000 条未读笔记的坟场。你需要的是像给朋友发短信一样简单的东西。

这个工作流将 OpenClaw 变成一个通过短信交互的记忆捕获系统，背后有一个可随时浏览的自定义可搜索界面。

## 功能介绍

- 通过 Telegram、iMessage 或 Discord 向你的 OpenClaw 发送任何内容——"提醒我读一本关于本地 LLM 的书"——它会立即记住
- OpenClaw 内置的记忆系统会永久存储你告诉它的一切
- 自定义的 Next.js 仪表盘让你可以搜索每一条记忆、对话和笔记
- 全局搜索（Cmd+K）覆盖所有记忆、文档和任务
- 无需文件夹、无需标签、无需复杂的组织——只需输入和搜索

## 痛点

每个笔记应用最终都会变成负担。你停止使用它，因为组织的摩擦比遗忘的摩擦更高。关键洞察是：**捕获应该像发短信一样简单，检索应该像搜索一样简单**。

## 所需技能

- Telegram、iMessage 或 Discord 集成（用于基于文本的捕获）
- Next.js（OpenClaw 会为你构建——无需编码）

## 设置方法

1. 确保你的 OpenClaw 已连接到你偏好的消息平台（Telegram、Discord 等）。

2. 立即开始使用——只需向你的机器人发送任何你想记住的内容：

以下是一些记忆捕获的示例：

```text
Hey, remind me to read "Designing Data-Intensive Applications"
Save this link: https://example.com/interesting-article
Remember: John recommended the restaurant on 5th street
```

3. 通过提示 OpenClaw 构建可搜索的界面：

以下提示词让智能体为你构建一个完整的第二大脑 Web 应用：

```text
I want to build a second brain system where I can review all our notes,
conversations, and memories. Please build that out with Next.js.

Include:
- A searchable list of all memories and conversations
- Global search (Cmd+K) across everything
- Ability to filter by date and type
- Clean, minimal UI
```

4. OpenClaw 会为你构建并部署整个 Next.js 应用。访问它提供的 URL，你就拥有了自己的第二大脑仪表盘。

5. 从现在开始，每当你想到什么——在路上、在会议中、睡前——只需给你的机器人发消息。需要查找什么的时候再回到仪表盘。

## 关键洞察

- 力量在于**零摩擦捕获**。你不需要打开应用、选择文件夹或添加标签。只需发消息。
- OpenClaw 的记忆系统是累积的——它记住你*曾经*告诉它的一切，使其随时间变得更加强大。
- 你可以从手机给机器人发消息，它会在你的电脑上构建东西。界面就是对话本身。

## 参考来源

灵感来自 [Alex Finn 关于改变生活的 OpenClaw 用例的视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)。

## 相关链接

- [OpenClaw 记忆系统](https://github.com/openclaw/openclaw)
- [打造第二大脑（Tiago Forte）](https://www.buildingasecondbrain.com/)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/second-brain.md)
