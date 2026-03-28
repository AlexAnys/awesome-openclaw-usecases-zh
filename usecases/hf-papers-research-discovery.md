# HuggingFace 论文发现与研究

> 含国内适配：HF 镜像站 / PaperWeekly / 飞书推送

跟进 ML（机器学习）前沿研究意味着每天刷 Hugging Face Papers 页面、扫几十篇标题、逐个点开看摘要、再手动交叉检索 GitHub 仓库。你需要一种对话式的方式来发现、筛选和深读热门论文，而不必离开工作区。

这个工作流组合两个技能（Skill），构建完整的研究发现流水线：

- 浏览当天 Hugging Face 热门论文——按点赞数或日期排序
- 按关键词搜索论文，快速定位任意主题的相关工作
- 获取完整论文元数据（Metadata）：摘要、作者、GitHub 仓库、社区点赞数、AI 生成摘要
- 阅读任意论文的社区讨论和评论
- 通过 arXiv ID 深读论文全文 LaTeX 源码（使用 arxiv-source 技能）

> **与 arXiv 论文阅读用例的关系**：本仓库的 [arXiv 论文阅读与 LaTeX 写作用例](https://github.com/AlexAnys/awesome-openclaw-usecases-zh/pull/36)聚焦于"已知论文的深度阅读和写作辅助"——给定 arXiv ID 后的全文解析、章节浏览、LaTeX 写作。本用例则聚焦于**上游环节：发现与筛选**——每日热门、关键词搜索、元数据浏览、社区信号。两者互补：先用本用例找到值得读的论文，再用 arXiv 用例深入阅读。

## 所需技能

- [hf-papers](https://github.com/openclaw/skills/tree/main/skills/willamhou/hf-papers) 技能（4 个工具：`hf_daily_papers`、`hf_search_papers`、`hf_paper_detail`、`hf_paper_comments`）
- [arxiv-source](https://github.com/openclaw/skills/tree/main/skills/willamhou/arxiv-source) 技能（3 个工具：`arxiv_fetch`、`arxiv_sections`、`arxiv_abstract`）——用于全文深读

无需 Docker（容器）或身份认证——两个技能均使用公开 API（应用编程接口），支持本地缓存（Local Caching）。

## 如何设置

1. 安装两个技能：

```bash
clawhub install hf-papers
clawhub install arxiv-source
```

2. 向 OpenClaw 发送以下提示词：

```text
I want to stay on top of ML research. Here's my daily workflow:

1. Every morning, show me the top 10 trending papers on Hugging Face (sorted by upvotes)
   - For each paper, show: title, upvotes, GitHub repo (if any), and 1-line AI summary

2. When I say "search [topic]":
   - Search HF Papers and show the most relevant results
   - Highlight papers with linked GitHub repos or high upvote counts

3. When I pick a paper (by ID):
   - Show the full abstract, authors, and linked resources
   - Show community comments if any
   - Ask if I want a deep read

4. For deep reads:
   - Fetch the full paper via arxiv-source
   - Summarize key contributions, methodology, and results
   - Note any linked code repos I should check out

Keep a running list of papers I've reviewed today with one-line takeaways.
```

提示词说明：
- 第 1 步：每天早上展示 HF 热门 Top 10，按点赞排序，附标题/点赞数/GitHub 仓库/AI 摘要
- 第 2 步：输入 "search [主题]" 进行关键词搜索，高亮有代码仓库或高点赞的论文
- 第 3 步：选定论文后展示完整摘要、作者、关联资源、社区评论
- 第 4 步：深读模式——通过 arxiv-source 获取全文，总结关键贡献/方法/结论
- 最后一行：维护今日已读论文列表，附一句话要点

3. 试一下："What's trending on Hugging Face Papers today?"

## 实用建议

- **筛选策略**：先看点赞数和评论数快速过滤，再看摘要决定是否深读。社区点赞数（Upvotes）是判断论文热度的有效信号
- **关键词组合**：搜索时尝试不同粒度的关键词，例如 "vision transformer" vs "ViT efficient inference"，缩小范围可以减少噪音
- **GitHub 仓库关联**：有关联 GitHub 仓库的论文通常意味着可复现代码已开源，优先关注这类论文
- **社区评论**：hf_paper_comments 工具可以获取社区讨论，有时评论中会有作者对方法局限性的补充说明
- **结合定时任务**：可以用 OpenClaw 的 cron job（定时任务）功能，每天早上自动推送热门论文摘要

## 中国用户适配

### HuggingFace 访问问题

HuggingFace 在中国大陆访问受限。以下是可用的替代方案：

| 方案 | 说明 |
|------|------|
| **hf-mirror.com** | 社区维护的 HF 镜像站，设置 `HF_ENDPOINT=https://hf-mirror.com` 即可 |
| **huggingface.ac.cn** | HF 官方中国合作镜像，包含 AI 论文精选页面 |
| 代理访问 | 自行配置网络代理访问原站 |

> **注意**：hf-papers 技能底层调用 HuggingFace 公开 API（`/api/daily_papers`、`/api/papers/search` 等）。如果你的网络无法直接访问 huggingface.co，需要在运行 OpenClaw 的环境中配置代理，或修改技能中的 API 基础 URL（Base URL）指向镜像站。

### 国内 AI 论文社区

除了 HuggingFace Papers，国内也有优质的 AI 论文发现渠道：

| 平台 | 特点 | 链接 |
|------|------|------|
| **PaperWeekly** | 学术论文推荐与解读，有知乎专栏和微信公众号 | [知乎](https://zhuanlan.zhihu.com/paperweekly) |
| **机器之心（Synced）** | AI 领域深度报道，含论文解读 | [官网](https://www.jiqizhixin.com/) |
| **智源社区（BAAI）** | 北京智源研究院社区，含论文速递 | [官网](https://hub.baai.ac.cn/) |
| **huggingface.ac.cn** | HF 中国镜像，每日论文精选页面 | [论文页](https://huggingface.ac.cn/papers) |

这些平台可以作为 HuggingFace Papers 的补充信息源，尤其在网络受限时提供替代发现渠道。

### 推送渠道适配

| 原版方案 | 国内替代 | 说明 |
|---------|---------|------|
| 终端对话 | **飞书机器人** | 支持富文本卡片消息，适合推送论文摘要 |
| 终端对话 | **钉钉群机器人** | Webhook（网络钩子）方式，配置最简单 |
| 终端对话 | **企业微信应用** | 企业用户首选 |

### 提示词适配

如果你更习惯中文交互，可以使用以下提示词：

```text
I want to track ML research daily. Here's my workflow:

1. Every morning at 9 AM, fetch the top 10 trending papers from Hugging Face Papers (sorted by upvotes)
   - Show: title, upvotes, GitHub repo link (if available), 1-line summary
   - Format as a numbered list for easy reference

2. After showing the list, wait for my instructions:
   - If I say a number (e.g. "3"), show the full details for that paper
   - If I say "search [keyword]", search for papers on that topic

3. For paper details, always include:
   - Full abstract
   - Author list
   - Linked GitHub repos
   - Community comments (if any)

4. When I say "deep read":
   - Use arxiv-source to fetch the full paper text
   - Summarize: key contributions, methodology, main results, limitations

5. At end of each session, compile a digest of all papers I reviewed today with one-line takeaways.

Respond in Chinese for summaries and explanations, but keep paper titles and technical terms in English.
```

提示词说明：
- 第 1 步：每天早 9 点获取 HF 热门 Top 10，以编号列表格式展示
- 第 2 步：输入数字查看详情，输入 "search [关键词]" 搜索
- 第 3 步：论文详情包含摘要、作者、GitHub 链接、社区评论
- 第 4 步：输入 "deep read" 启动深读模式
- 第 5 步：每次会话结束时汇总今日已读论文
- 最后一行：摘要和说明用中文回复，论文标题和术语保留英文

## 相关链接

- [hf-papers 技能](https://github.com/openclaw/skills/tree/main/skills/willamhou/hf-papers) — HuggingFace 论文浏览与搜索
- [arxiv-source 技能](https://github.com/openclaw/skills/tree/main/skills/willamhou/arxiv-source) — arXiv 论文全文获取
- [HuggingFace Daily Papers](https://huggingface.co/papers) — 每日 AI 论文精选
- [HF Papers API 文档（非官方）](https://github.com/0x0is1/hf-papers-api-docs) — HuggingFace Papers API 端点参考
- [huggingface.ac.cn 论文页](https://huggingface.ac.cn/papers) — HF 中国镜像论文精选

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/hf-papers-research-discovery.md)
