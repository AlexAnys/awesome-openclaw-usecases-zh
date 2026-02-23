# 语义记忆搜索

OpenClaw 的内置记忆系统将所有内容存储为 Markdown 文件——但随着记忆在数周和数月间不断增长，要找到上周二的那个决定变得几乎不可能。没有搜索功能，只能滚动浏览文件。

这个用例使用 [memsearch](https://github.com/zilliztech/memsearch) 在 OpenClaw 现有的 Markdown 记忆文件之上添加了**向量驱动的语义搜索**，让你可以通过语义而非仅靠关键词即时找到任何过去的记忆。

## 功能介绍

- 用一条命令将所有 OpenClaw Markdown 记忆文件索引到向量数据库（Milvus）中
- 按语义搜索："我们选了什么缓存方案？"即使文档中没有出现"缓存"这个词也能找到相关记忆
- 混合搜索（dense vector（稠密向量）+ BM25 全文检索）配合 RRF（倒数排名融合）重排序，获得最佳结果
- SHA-256 内容哈希意味着未更改的文件永远不会被重新嵌入——零 API 调用浪费
- 文件监视器在记忆文件变更时自动重新索引，保持索引始终最新
- 支持任何 embedding（向量嵌入）提供商：OpenAI、Google、Voyage、Ollama，或完全本地运行（无需 API 密钥）

## 痛点

OpenClaw 的记忆以纯 Markdown 文件存储。这对可移植性和人类可读性来说很好，但没有搜索功能。随着记忆增长，你要么只能 grep 文件（仅支持关键词，会漏掉语义匹配），要么将整个文件加载到上下文中（在无关内容上浪费 token）。你需要一种方式来问"我关于 X 做了什么决定？"并获得精确的相关片段，而不受措辞影响。

## 所需技能

- 无需 OpenClaw 技能——memsearch 是一个独立的 Python CLI/库
- Python 3.10+ 以及 pip 或 uv

## 如何设置

1. 安装 memsearch：
```bash
pip install memsearch
```

2. 运行交互式配置向导：
```bash
memsearch config init
```

3. 索引你的 OpenClaw 记忆目录：
```bash
memsearch index ~/path/to/your/memory/
```

4. 按语义搜索你的记忆：
```bash
memsearch search "what caching solution did we pick?"
```

5. 若需实时同步，启动文件监视器——每次文件变更时自动索引：
```bash
memsearch watch ~/path/to/your/memory/
```

6. 若需完全本地化部署（无需 API 密钥），安装本地 embedding 提供商：
```bash
pip install "memsearch[local]"
memsearch config set embedding.provider local
memsearch index ~/path/to/your/memory/
```

## 关键洞察

- **Markdown 始终是数据源。** 向量索引只是一个派生缓存——你可以随时用 `memsearch index` 重建它。你的记忆文件永远不会被修改。
- **智能去重节省成本。** 每个文本块通过 SHA-256 内容哈希标识。重新运行 `index` 只会嵌入新增或更改的内容，因此你可以随意运行而不会浪费 embedding API 调用。
- **混合搜索优于纯向量搜索。** 通过 Reciprocal Rank Fusion（倒数排名融合）将语义相似度（稠密向量）与关键词匹配（BM25）结合，既能捕获基于语义的查询，也能捕获精确匹配的查询。

## 相关链接

- [memsearch GitHub](https://github.com/zilliztech/memsearch) —— 驱动此用例的库
- [memsearch 文档](https://zilliztech.github.io/memsearch/) —— 完整的 CLI 参考、Python API 和架构说明
- [OpenClaw](https://github.com/openclaw/openclaw) —— 启发 memsearch 的记忆架构
- [Milvus](https://milvus.io/) —— 向量数据库后端

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/semantic-memory-search.md)
