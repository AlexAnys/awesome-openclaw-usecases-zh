# 多智能体内容工厂

你是一名内容创作者，需要在多个平台上兼顾调研、写作和设计。每一个步骤——寻找热门话题、撰写脚本、生成缩略图——都要耗费数小时。如果一个由专业智能体（agent）组成的团队能在一夜之间处理这一切呢？

这个工作流在 Discord 内建立一个多智能体（multi-agent）内容工厂，不同的智能体在专属频道中分别负责调研、写作和视觉素材。

## 功能概述

- **调研智能体**每天早上扫描热门故事、竞品内容和社交媒体，找到最佳内容机会
- **写作智能体**选取最佳创意，撰写完整的脚本、推文串或新闻简报草稿
- **缩略图智能体**为内容生成 AI 缩略图或封面图片
- 每个智能体在自己的 Discord 频道中工作，保持一切有条理且可审阅
- 按计划自动运行（例如每天早上 8 点），让你醒来就看到完成的内容

## 痛点

内容创作有三个阶段——调研、写作和设计——大多数创作者都在手动完成这三个阶段。即使使用 AI 写作工具，你仍然需要逐一提示它们。这个系统将智能体链接成一条管线，一个智能体的输出作为下一个的输入，完全无需人工干预。

## 所需技能

- Discord 集成，配置多个频道
- `sessions_spawn` / `sessions_send`，用于多智能体编排
- [x-research-v2](https://clawhub.ai) 或类似工具，用于社交媒体调研
- 本地图像生成（如 Nano Banana）或图像生成 API
- [knowledge-base](https://clawhub.ai) 技能（可选，用于 RAG 驱动的调研）

## 如何设置

1. 设置一个 Discord 服务器（或者让 OpenClaw 帮你设置——只需说"帮我建一个 Discord"）。

2. 为每个智能体创建频道：
   - `#research` — 热门话题和内容机会
   - `#scripts` — 撰写的草稿和大纲
   - `#thumbnails` — 生成的图片和封面

3. 提示 OpenClaw：

```text
I want you to build me a content factory inside of Discord.
Set up channels for different agents:

1. Research Agent (#research): Every morning at 8 AM, research top trending
   stories, competitor content, and what's performing well on social media
   in my niche. Post the top 5 content opportunities with sources.

2. Writing Agent (#scripts): Take the best idea from the research agent
   and write a full script/thread/newsletter draft. Post it in #scripts.

3. Thumbnail Agent (#thumbnails): Generate AI thumbnails or cover images
   for the content. Post them in #thumbnails.

Have all their work organized in different channels.
Run this pipeline automatically every morning.
```

4. 根据你的平台进行自定义：

提示 OpenClaw：

```text
I focus on X/Twitter threads, not YouTube. Change the writing agent
to produce tweet threads instead of video scripts.
```

## 关键洞察

- 核心在于**链式智能体**——调研驱动写作，写作驱动缩略图。你无需逐步单独提示。
- Discord 频道让你可以轻松地分别审阅每个智能体的工作，并提供反馈，如"脚本太长了"或"多关注 AI 新闻"。
- 你可以将此模式适配到任何内容格式：推文、新闻简报、LinkedIn 帖子、播客大纲、博客文章。
- 使用本地模型进行图像生成（如在 Mac Studio 上运行 Nano Banana）可以降低成本并获得更多控制权。

## 灵感来源

受 [Alex Finn 关于改变生活的 OpenClaw 用例视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)启发。

## 相关链接

- [OpenClaw 子智能体文档](https://github.com/openclaw/openclaw)
- [Discord Bot 设置指南](https://discord.com/developers/docs)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/content-factory.md)
