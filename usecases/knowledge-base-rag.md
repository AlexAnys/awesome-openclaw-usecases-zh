# 个人知识库（RAG）

你整天都在阅读文章、推文和观看视频，但总是找不到上周看到的那个东西。书签越积越多，最终变得毫无用处。

这个工作流从你保存的所有内容中构建一个可搜索的知识库：

- 将任何 URL 放入 Telegram 或 Slack，系统自动摄入内容（文章、推文、YouTube 字幕、PDF）
- 对你保存的所有内容进行语义搜索：搜索"我保存了哪些关于 agent memory 的内容？"会返回带来源的排序结果
- 可以与其他工作流联动——例如，视频创意流水线在构建调研卡片时会查询知识库中的相关已保存内容

## 所需技能

- [knowledge-base](https://clawhub.ai) 技能（或使用 embedding（向量嵌入）自建 RAG（检索增强生成）系统）
- `web_fetch`（内置）
- 用于内容摄入的 Telegram 话题或 Slack 频道

## 如何设置

1. 从 ClawdHub 安装 knowledge-base 技能。
2. 创建一个名为 "knowledge-base" 的 Telegram 话题（或使用 Slack 频道）。
3. 向 OpenClaw 发送以下提示词：
```text
When I drop a URL in the "knowledge-base" topic:
1. Fetch the content (article, tweet, YouTube transcript, PDF)
2. Ingest it into the knowledge base with metadata (title, URL, date, type)
3. Reply with confirmation: what was ingested and chunk count

When I ask a question in this topic:
1. Search the knowledge base semantically
2. Return top results with sources and relevant excerpts
3. If no good matches, tell me

Also: when other workflows need research (e.g., video ideas, meeting prep), automatically query the knowledge base for relevant saved content.
```

4. 测试一下：放入几个 URL，然后提问，例如"我有哪些关于 LLM memory 的内容？"

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/knowledge-base-rag.md)
