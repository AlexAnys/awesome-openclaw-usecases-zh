# 目标驱动的自主任务系统

你的 AI 智能体（agent）功能强大，但本质上是被动的——只有你下达指令时它才会工作。如果它能了解你的目标，并每天主动提出推进目标的任务，而无需你开口呢？

这个工作流将 OpenClaw 变成一个自主驱动的"员工"。你只需一次性倾倒你的所有目标，智能体就会自主生成、安排并完成推进这些目标的任务——甚至会在一夜之间为你构建惊喜小应用。

## 功能概述

- 你将所有目标、使命和任务（个人和职业）一次性倾倒给 OpenClaw
- 每天早上，智能体会自动生成 4-5 个可以在你的电脑上自主完成的任务
- 任务不仅限于构建应用：还包括调研、编写脚本、开发功能、创建内容、分析竞品等
- 智能体自行执行任务，并在它为你构建的自定义看板（Kanban board）上跟踪进度
- 你还可以让它每晚为你构建一个惊喜小应用——一个新的 SaaS 创意、一个自动化你生活中无聊部分的工具，以 MVP（最小可行产品）的形式交付

## 痛点

大多数人有远大的目标，但难以将其分解为每日可执行的步骤。即使做到了，执行也会耗尽所有时间。这个系统将计划和执行都交给了你的 AI 智能体。你定义目的地，智能体规划每日路径并逐步执行。

## 所需技能

- Telegram 或 Discord 集成
- `sessions_spawn` / `sessions_send`，用于自主任务执行
- Next.js 或类似框架（用于看板——OpenClaw 会为你构建）

## 如何设置

### 第一步：倾倒你的目标

这是最重要的一步。把你想要实现的一切告诉 OpenClaw：

提示 OpenClaw：

```text
Here are my goals and missions. Remember all of this:

Career:
- Grow my YouTube channel to 100k subscribers
- Launch my SaaS product by Q3
- Build a community around AI education

Personal:
- Read 2 books per month
- Learn Spanish

Business:
- Scale revenue to $10k/month
- Build partnerships with 5 companies in my space
- Automate as much of my workflow as possible

Use this context for everything you do going forward.
```

### 第二步：设置自主每日任务

提示 OpenClaw：

```text
Every morning at 8:00 AM, come up with 4-5 tasks that you can complete
on my computer today that bring me closer to my goals.

Then schedule and complete those tasks yourself. Examples:
- Research competitors and write analysis reports
- Draft video scripts based on trending topics
- Build new features for my apps
- Write and schedule social media content
- Research potential business partnerships
- Build me a surprise mini-app MVP that gets me closer to one of my goals

Track all tasks on a Kanban board. Update the board as you complete them.
```

### 第三步：构建看板（可选）

提示 OpenClaw：

```text
Build me a Kanban board in Next.js where I can see all the tasks you're
working on. Show columns for To Do, In Progress, and Done. Update it
in real-time as you complete tasks.
```

## 关键洞察

- **目标倾倒是一切的基础**。你提供的目标上下文越丰富，智能体生成的每日任务就越好。不要有所保留。
- 智能体会发现你自己想不到的任务。它会在你的各个目标之间建立联系，找到你可能错过的机会。
- 看板将你的智能体变成一个可追踪的"员工"。你可以准确看到它一直在做什么，并及时调整方向。
- 对于夜间应用构建：明确告诉它构建 MVP，不要过度复杂化。你每天早上醒来都会收到一个新惊喜。
- 这个效果会随时间复利增长——智能体会学习哪类任务最有帮助，并自动调整。

## 灵感来源

受 [Alex Finn](https://www.youtube.com/watch?v=UTCi_q6iuCM&t=414s) 及其[关于改变生活的 OpenClaw 用例视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)启发。

## 相关链接

- [OpenClaw 记忆系统](https://github.com/openclaw/openclaw)
- [OpenClaw 子智能体文档](https://github.com/openclaw/openclaw)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/overnight-mini-app-builder.md)
