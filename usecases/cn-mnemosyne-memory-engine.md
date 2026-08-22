# 用 Mnemosyne 给 AI 助手装上"永不失忆"的认知记忆

> 🇨🇳 国内生态原创用例 · 零依赖 · 零 API 成本 · 数据 100% 本地

## 解决什么问题

OpenClaw（以及所有对话式 AI 助手）有一个共同的痛点：**"失忆"**。

每次新会话，它就忘了你是谁、你上次聊了什么、你喜欢什么。你用得越久，这个割裂感越明显——明明是个"私人助手"，却像每天失忆一次。

社区常见的解决方案是接 Mem0、Zep、LangChain Memory 这类记忆服务，但它们都需要：
- 调用 LLM API（产生 token 成本）
- 部署向量数据库（Milvus/PostgreSQL+pgvector）
- 配置 embedding 模型

对国内用户来说还多一层问题：**数据要离开本机**，不符合个人信息保护的要求。

Mnemosyne 走了一条完全不同的路：**纯本地、零神经网络、纯 Markdown 文件的认知记忆引擎**。

## 为什么它适合国内用户

| 特性 | 对国内用户的意义 |
|---|---|
| 零 API 调用 | 记忆检索不产生任何 token 成本，用多久都免费 |
| 零向量数据库 | 不需要部署 Milvus/PG，普通电脑就能跑 |
| 数据 100% 本地 | 所有记忆就是 Markdown 文件，可以 `git diff`，不离开你的机器 |
| 支持阿里云百炼 | 对话模型可以接国内可用的百炼（Qwen），无需翻墙 |
| 适配国产大模型 | 与通义千问深度测试，中文场景效果好 |

## 架构原理（一句话）

不用神经网络，用**认知心理学**。基于 140 年记忆研究（艾宾浩斯遗忘曲线 1885 → SAM 复合线索理论 1981 → 蔡格尼克效应 1927），把心理学公式翻译成代码。

核心公式：
```
熟悉度 = 0.35×重要性 + 0.25×时效性 + 0.25×关键词 + 0.10×命中频率 + 0.05×层级权重
```

每个权重都能追溯到一篇具名论文，不是调参调出来的。

## 安装步骤（5 分钟）

### 1. 下载

```bash
git clone https://github.com/ElonAug7/Mnemosyne-agentmemory-engine-openclaw-hermes.git
cd Mnemosyne-agentmemory-engine-openclaw-hermes/Mnemosyne-v6.4
```

### 2. 一键安装

```bash
bash install.sh
```

安装会自动：
- 初始化四层记忆目录（短期/工作台/中期/长期）
- 把记忆协议写入 `SOUL.md` 和 `AGENTS.md`
- 启动 Web UI（只监听 `127.0.0.1`，自动带 token）

### 3. 打开记忆面板

浏览器访问：`http://127.0.0.1:8765`

## 验证记忆生效（可复现步骤）

### 第一轮对话（让它记住）

告诉你的助手任何个人信息，比如：

> "我叫小王，在杭州做前端开发，最近在学 Rust"

### 重启会话后（新对话）

直接问：

> "你还记得我是谁吗？我在学什么？"

如果记忆生效，它会引用之前记录的内容回答你，而不是说"我不知道"。

### 用命令直接查记忆

```bash
# 查看你的画像（引擎自动提炼的技术栈、偏好、风格）
node engine.js profile

# 搜索记忆
node engine.js search --query "Rust 学习" --mode keyword

# 查看健康状态
node engine.js status
```

## 实测数据

在 Memory-Native Evaluation 基准（80 查询，11 个系统）上：

| 系统 | nDCG@10 |
|---|---|
| Mnemosyne v6.2 | 0.046 |
| 裸 BM25 基线 | 0.185 |
| 嵌入系统（Mem0 等） | 0.12–0.16 |
| **Mnemosyne v6.4** | **0.238** |

——**用零 embedding 反超了所有嵌入系统**，搜索延迟约 7ms。

## 难度

⭐⭐（需要命令行基础，会 `git clone` 和 `bash install.sh` 即可）

## 适合谁

- 想让 AI 助手真正"认识你"的个人用户
- 对数据隐私敏感、不想把聊天记录传到云端的用户
- 想零成本跑长期记忆、不想付 token 费的用户
- 对认知科学/记忆机制感兴趣的开发者

## 参考链接

- 开源仓库：[ElonAug7/Mnemosyne-agentmemory-engine-openclaw-hermes](https://github.com/ElonAug7/Mnemosyne-agentmemory-engine-openclaw-hermes)
- 完整技术文档：仓库内 `MNEMOSYNE-REFERENCE.md`
- 版本历史：仓库内 `CHANGELOG.md`
