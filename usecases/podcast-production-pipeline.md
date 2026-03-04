# 播客制作流水线

> 翻译自 [awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)，含国内适配（小宇宙 / 喜马拉雅 / B站 / 小红书宣发）。

你有一堆播客选题，甚至已经列好了待录清单。但在嘉宾调研、大纲撰写、开场白起草、节目笔记整理、社交媒体宣发之间，制作开销消耗了你所有的创作热情。如果你只需要给出一个话题，就能拿回一整套可发布的制作包呢？

这个用例把多个智能体串联起来，处理从选题到发布的完整播客制作流程。

## 痛点

独立播客主播和小团队在制作上花的时间远多于录制本身。嘉宾调研动辄数小时，节目笔记总是被拖延，社交媒体宣发更是第一个被砍掉的环节。真正有创造力的部分——对话本身——可能只占总工作量的 30%。这个智能体帮你处理剩下的 70%。

## 它能做什么

- **节目调研** — 给出话题或嘉宾名字，自动整理背景资料、谈话要点和建议问题
- **大纲与脚本** — 生成结构化的节目大纲，包括开场白脚本、段落过渡和结束语
- **节目笔记** — 录制完成后，将转录稿处理成带时间戳的节目笔记，附上提到的所有链接
- **社交媒体素材包** — 为 X、LinkedIn 和 Instagram 生成包含节目亮点和金句的宣发帖文
- **节目描述** — 撰写针对 Spotify、Apple Podcasts 和 YouTube 优化的 SEO 节目描述

## 所需技能

- 网络搜索 / 调研技能（用于嘉宾调研和话题深度研究）
- 文件系统访问（用于读取转录稿和写入输出文件）
- Slack、Discord 或 Telegram 集成（用于交付成品）
- 可选：`sessions_spawn`（用于并行运行调研和写作智能体）
- 可选：RSS feed 技能（用于监控竞品播客）

## 如何设置

1. 录制前——生成调研资料和节目大纲：

```text
I'm recording a podcast episode about [TOPIC]. My guest is [NAME].

Please:
1. Research the guest — their background, recent work, hot takes, and
   anything controversial or interesting they've said publicly.
2. Research the topic — key trends, recent news, common misconceptions,
   and what the audience likely already knows vs. what would surprise them.
3. Generate an episode outline:
   - Cold open hook (1-2 sentences to grab attention)
   - Intro script (30 seconds, casual tone)
   - 5-7 interview questions, ordered from easy/rapport-building to deep/provocative
   - 2-3 "back pocket" questions in case the conversation stalls
   - Closing segment with call-to-action

Save everything to ~/podcast/episodes/[episode-number]/prep/
```

2. 录制后——生成节目笔记和宣发素材：

```text
Here's the transcript for Episode [NUMBER]: [paste or point to file]

Please:
1. Write timestamped show notes — every major topic shift gets a timestamp
   and one-line summary. Include links to anything mentioned (tools, books,
   articles, people).
2. Write an episode description (max 200 words) optimized for podcast
   search. Include 3-5 relevant keywords naturally.
3. Create social media posts:
   - X/Twitter: 3 tweets — one pull quote, one key insight, one question
     to spark discussion. Each under 280 chars.
   - LinkedIn: 1 post, professional tone, 100-150 words.
   - Instagram caption: 1 post with emoji, casual tone, include relevant hashtags.
4. Extract a "highlights" list — the 3 most interesting/surprising moments
   with timestamps.

Save everything to ~/podcast/episodes/[episode-number]/publish/
```

3. 可选——竞品播客监控：

```text
Monitor these podcast RSS feeds daily:
- [feed URL 1]
- [feed URL 2]

When a new episode drops that covers a topic relevant to my podcast,
send me a Telegram message with:
- Episode title and link
- One-sentence summary
- Whether this is something I should respond to or cover from my angle
```

## 关键洞察

- **录制前调研**是最大的价值所在。带着深度嘉宾调研走进录音棚，对话质量会有质的飞跃——这不是后期能弥补的。
- 带时间戳的**节目笔记**是提升听众留存的利器。大多数播客主播因为嫌麻烦而跳过这一步，但智能体让它变得毫不费力。
- **社交媒体素材包**节省的是最多的*持续性*时间。每期都需要宣发，结构完全一样——这正是自动化的完美场景。
- 与[多智能体内容工厂](content-factory.md)搭配使用效果更佳，可以将播客内容转化为博客文章、Newsletter 或视频片段。

## 中国用户适配

国内播客生态与海外有显著差异，以下是针对中国播客创作者的适配建议。

### 平台替代方案

| 海外平台 | 国内替代 | 说明 |
|---------|---------|------|
| Spotify for Podcasters | 小宇宙 | 国内最活跃的纯播客平台，支持 RSS 订阅和托管，有完善的创作者后台 |
| Apple Podcasts | 喜马拉雅 | 用户体量最大的音频平台，需单独上传（不支持纯 RSS 导入） |
| YouTube (播客视频化) | 网易云音乐播客 / B 站 | 网易云支持 RSS 导入；B 站适合视频播客 |
| Anchor | 荔枝 FM / 蜻蜓 FM | 可作为补充分发渠道 |
| Slack/Discord | 飞书 / 钉钉 / 微信 | 用于接收智能体交付的制作包 |

### 分发策略

国内播客分发分为两类：

- **RSS 分发**（小宇宙、Apple Podcasts、网易云音乐等泛用型客户端）：生成一条 RSS 链接，提交到各平台即可自动同步
- **手动上传**（喜马拉雅、荔枝 FM 等）：需要单独上传音频文件和节目信息，可以让智能体准备好各平台所需的元数据（标题、描述、标签），减少重复劳动

建议使用 Firstory 等托管平台自动生成 RSS，再手动补充需要单独上传的平台。

### 制作工具适配

- **转录**：录制完成后可使用 [Whisper](https://github.com/openai/whisper) 进行本地转录，也可使用讯飞听见、飞书妙记等国内服务，中文识别准确率更高
- **剪辑**：小宇宙提供「小宇宙 Studio」在线剪辑工具，与主播后台深度集成；也可使用 Adobe Audition、达芬奇等专业工具
- **封面和视觉素材**：配合 AI 图像生成工具制作每期封面，保持品牌一致性

### 社交媒体宣发适配

录制后的宣发提示词建议替换为国内平台：

```text
Here's the transcript for Episode [NUMBER]: [paste or point to file]

Please:
1. Write timestamped show notes (same as original).
2. Write an episode description (max 200 words) with 3-5 keywords,
   optimized for 小宇宙 and 喜马拉雅 search.
3. Create social media posts:
   - 小红书: 1 post, conversational tone, include 5-8 hashtags,
     highlight the most relatable listener moment.
   - 微信公众号: 1 article draft, 800-1200 words, deeper recap
     of episode themes with key quotes.
   - 微博: 2 posts — one pull quote with guest photo prompt,
     one question to spark discussion. Each under 140 chars.
4. Extract a "highlights" list — the 3 most interesting moments
   with timestamps, formatted for short video clips (抖音/视频号).

Save everything to ~/podcast/episodes/[episode-number]/publish/
```

### 竞品监控适配

将 RSS 监控目标替换为你关注的国内播客。小宇宙上的播客大多提供 RSS 链接，可以在节目页面找到。

### 国内创作者特别提醒

- **中文播客制作周期长**：据 CPA 中文播客社区统计，2024 年中文播客创作者平均每期净工作时长达 12.9 小时，其中剪辑平均耗时 4.5 小时。这意味着从调研、节目笔记到宣发的自动化可以显著降低非核心工作负担
- **兼职创作者居多**：近八成播客从业者为兼职状态，制作时间碎片化。用智能体自动化流水线工作，可以把有限的时间集中在内容本身
- **视频播客趋势**：2026 年视频播客正从"补充形态"变为"主流形态"。考虑在流水线中加入短视频切片（抖音/视频号）的自动化环节

## 相关链接

- [Podcast RSS Feed Spec (Apple)](https://podcasters.apple.com/support/823-podcast-requirements)
- [Spotify for Podcasters](https://podcasters.spotify.com/)
- [Whisper (OpenAI)](https://github.com/openai/whisper) — 本地转录生成
- [小宇宙主播入驻](https://podcaster.xiaoyuzhoufm.com/)
- [喜马拉雅创作中心](https://zhubo.ximalaya.com/)

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/podcast-production-pipeline.md)
