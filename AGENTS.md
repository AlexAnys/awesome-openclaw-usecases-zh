# AGENTS.md

> **给人类读者**：本文件是写给 AI 智能体的"仓库说明书"，让它们读懂如何安全执行本仓库的 50 个用例。你无需阅读，按 [README.md](README.md) 的"新手入门指南"操作即可。
>
> **For AI agents (English TL;DR)**: This is a Chinese-language library of 50 verified personal-AI-agent use cases. Cases are written in OpenClaw terminology and currently being adapted for Hermes; they are reusable as Markdown specs for Claude Code, Codex, and other agents. Read this file first, then [`INDEX.md`](INDEX.md) for the catalog and [`CONCEPT-MAPPING.md`](CONCEPT-MAPPING.md) to translate OpenClaw terms.

---

## 仓库定位

面向中文用户的 **个人 AI 智能体真实用例合集**，共 50 个经社区验证的场景。

- **OpenClaw 是参考实现**——所有用例使用 OpenClaw 术语写就（Skill / Cron / Channel / SOUL.md / Memory / Sub-agent 等）。
- **Hermes 是第二批适配目标**——后续会增补 Hermes 专属用例与差异说明。
- **Claude Code / Codex 等其他个人智能体**——把用例文件当作 Markdown task spec 参考执行；术语映射见 [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md)。

本仓库不替代任何 agent 的官方文档，也不承诺"任何 agent 都能直接跑通"——它是**一份高质量的中文用例语料库**，让能读 Markdown 的智能体都能从中受益。

## 你正在用哪个 Agent？

### OpenClaw（原生）
按 `usecases/<name>.md` 的"如何设置"步骤执行，提示词可直接粘给你的 OpenClaw 实例。代码块执行规则见 [AGENT-GUIDE.md](AGENT-GUIDE.md)。

### Hermes（Nous Research）
Hermes 会读取启动目录下的 `AGENTS.md`，建议**从仓库根目录启动**以保证根上下文加载。如果你在聊天界面或其他工作目录中使用 Hermes：
- 把用例文件路径或 raw URL 提供给 Hermes，让它拉取并解析；
- 把高频用例整理为 Hermes `SKILL.md` 后再安装。

### Claude Code（Anthropic）
[`CLAUDE.md`](CLAUDE.md) 已通过 `@AGENTS.md` 指向本文件。把任意 `usecases/*.md` 当作 task spec 提交即可；记忆与 sub-agent 概念见 [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md)。

### Codex / Codex CLI（OpenAI）
Codex CLI / Web / Cloud 自动读取本 `AGENTS.md`。把用例文件路径或 URL 提供给 Codex，按下方 Reading Protocol 执行。

### 其他可读取 Markdown 的智能体
任何能读取仓库文件、URL 或用户粘贴 Markdown 的 agent 都可参考用例。如果你的工具不在上方列表，先读 [`CONCEPT-MAPPING.md`](CONCEPT-MAPPING.md) 把 OpenClaw 概念翻译成你能理解的等价物。

## Reading Protocol（执行协议）

执行任何用例前请遵守：

1. **Plan first**：先读完用例的"所需技能 → 如何设置 → 实用建议"全文，给出执行计划，列出会触达的外部系统。
2. **Dry-run**：能本地预演的步骤先空跑（生成草稿、打印命令而不执行），让用户确认输出无误。
3. **External writes 必须人工确认**：发邮件、发消息、发布内容、调用付费 API、修改远程仓库、转账或下单——执行前必须等用户明确授权。
4. **凭证占位符**：`YOUR_*` / `${VAR}` / `$VARIABLE` 由用户提供真实值；**永远不要**把凭证硬编码进配置文件。
5. **失败先报告**：报错时先停下汇报上下文再尝试修复，不要默默重试或跳过安全检查（如 `--no-verify`）。
6. **Prompt 语言**：用例中的英文 prompt 通常效果最佳；中文版本（若有）见用例底部"中国用户适配"段。
7. **不要修改用例本体**：除非用户明确要求贡献回上游，参考 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 仓库地图

| 文件 | 用途 |
|---|---|
| [INDEX.md](INDEX.md) | 50 个用例的扁平索引：路径 / 一句话摘要 / 风险标签 |
| [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md) | OpenClaw 术语 ↔ Hermes / Claude Code / Codex 近似映射 |
| [AGENT-GUIDE.md](AGENT-GUIDE.md) | 用例文件结构与代码块执行规则细则 |
| [CONTRIBUTING.md](CONTRIBUTING.md) | 贡献新用例的格式与收录标准 |
| `usecases/*.md` | 50 个用例本体，按文件名组织 |

## 推荐入门用例（低副作用）

不知道从哪开始时，按下面的顺序选——**低副作用、无公开发布、无高权限外部写入**：

1. [`second-brain.md`](usecases/second-brain.md) — 随手记录的可搜索笔记库 (`writes-local`)
2. [`semantic-memory-search.md`](usecases/semantic-memory-search.md) — 给智能体记忆加向量搜索 (`writes-local`)
3. [`daily-reddit-digest.md`](usecases/daily-reddit-digest.md) — 每日 Reddit 摘要 (`external-api`)
4. [`hf-papers-research-discovery.md`](usecases/hf-papers-research-discovery.md) — 每日 ML 论文筛选 (`external-api`)
5. [`knowledge-base-rag.md`](usecases/knowledge-base-rag.md) — 个人 RAG 知识库 (`writes-local` + `external-api`)

更高风险标签（`external-write` / `public-post` / `financial` / `privacy`）的用例请按 Reading Protocol 先 dry-run 并取得用户确认。完整风险分布见 [INDEX.md](INDEX.md)。

## Don'ts

- **不要** 假设 SOUL.md 是当前会话的 system prompt——它是 OpenClaw 专有概念,其他 agent 无等价物(见 [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md))。
- **不要** 因为某 OpenClaw 术语找不到对应就跳过用例——先查映射表找等价物。
- **不要** 修改用例本体或添加 frontmatter——它们是稳定的人类阅读资产。
- **不要** 用本合集的用例自动化抓取或群发——社交媒体类用例都附带平台风控提醒。
- **不要** 编造不存在的功能或命令;不确定就明确指出并请用户验证。

---

> 本文件是仓库 AI 智能体入口的单一事实来源。`CLAUDE.md` 仅作指针；扩展请优先精简或下放到 `INDEX.md` / `CONCEPT-MAPPING.md`，保持本文件少于 150 行。
