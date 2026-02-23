# YouTube 内容管线

作为一名日更的 YouTube 创作者，在网络和 X/Twitter 上寻找新鲜、及时的视频创意非常耗时。追踪已经覆盖过的内容可以避免重复，并帮助你保持领先趋势。

这个工作流自动化了整个内容搜集和调研管线：

- 每小时定时任务（cron job）扫描突发 AI 新闻（网络 + X/Twitter），并将视频创意推送到 Telegram
- 维护一个 90 天的视频目录，包含播放量和主题分析，避免重复覆盖话题
- 将所有创意存储在 SQLite 数据库中，并使用向量嵌入（vector embeddings）进行语义去重（这样你永远不会收到重复的创意推荐）
- 当你在 Slack 中分享一个链接时，OpenClaw 会调研该话题、在 X 上搜索相关帖子、查询你的知识库，并在 Asana 中创建一个包含完整大纲的任务卡

## 所需技能

- `web_search`（内置）
- [x-research-v2](https://clawhub.ai) 或自定义 X/Twitter 搜索技能
- [knowledge-base](https://clawhub.ai) 技能，用于 RAG（检索增强生成）
- Asana 集成（或 Todoist）
- `gog` CLI，用于 YouTube Analytics 数据
- Telegram 话题，用于接收创意推送

## 如何设置

1. 在 Telegram 中设置一个视频创意话题，并在 OpenClaw 中配置。
2. 安装 knowledge-base 技能和 x-research 技能。
3. 创建用于创意追踪的 SQLite 数据库：

```sql
CREATE TABLE pitches (
  id INTEGER PRIMARY KEY,
  timestamp TEXT,
  topic TEXT,
  embedding BLOB,
  sources TEXT
);
```

4. 提示 OpenClaw：

```text
Run an hourly cron job to:
1. Search web and X/Twitter for breaking AI news
2. Check against my 90-day YouTube catalog (fetch from YouTube Analytics via gog)
3. Check semantic similarity against all past pitches in the database
4. If novel, pitch the idea to my Telegram "video ideas" topic with sources

Also: when I share a link in Slack #ai_trends, automatically:
1. Research the topic
2. Search X for related posts
3. Query my knowledge base
4. Create an Asana card in Video Pipeline with a full outline
```

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/youtube-content-pipeline.md)
