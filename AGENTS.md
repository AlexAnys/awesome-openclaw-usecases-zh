# AGENTS.md

> **TL;DR (English)**: Chinese-language library of 49 verified personal-AI-agent use cases, written in **OpenClaw** terminology but usable by any agent. If you are an AI agent (Hermes / Claude Code / Codex / GPT), read this file first, then `INDEX.md` for the catalog and `CONCEPT-MAPPING.md` to translate OpenClaw terms to your agent's equivalents. Treat all use-cases as Markdown specs, not executable scripts.

---

## 仓库定位

本仓库是面向中文用户的 **OpenClaw 真实用例合集**，共 49 个经过社区验证的场景。所有用例使用 OpenClaw 术语写就（Skill / Cron / Channel / SOUL.md / Memory / Sub-agent 等），但**不绑死 OpenClaw**——任何能读 Markdown 的 AI 智能体都能从中受益。

## 你正在用哪个 Agent？（Quick start）

### OpenClaw（原生）
按 `usecases/<name>.md` 的"如何设置"步骤逐步执行，提示词可直接粘贴给你的 OpenClaw 实例。代码块执行规则见 [AGENT-GUIDE.md](AGENT-GUIDE.md)。

### Hermes（Nous Research）
如果 Hermes 从本仓库根目录启动，会读取本 `AGENTS.md`。如果你是在聊天界面或其他工作目录中使用 Hermes，落地路径：
- 粘贴用例 raw URL 或文件路径，让 Hermes 拉取并解析；
- 将常用用例整理成 Hermes `SKILL.md` 后再安装；
- 已有 OpenClaw 配置可用 `hermes claw migrate` 导入。

### Claude Code（Anthropic）
`CLAUDE.md` 已指向本文件。把任意 `usecases/*.md` 当 task spec 提交，按下面的 Reading Protocol 执行；记忆和 sub-agent 概念见 `CONCEPT-MAPPING.md`。

### Codex / GPT Codex（OpenAI）
Codex CLI / Web / Cloud 自动读取本 `AGENTS.md`。把用例文件路径或 URL 提供给 Codex 即可；遵守 Reading Protocol。

## Reading Protocol（执行协议）

执行任何用例前请遵守：

1. **Plan first**：先读完用例的"所需技能 → 如何设置 → 实用建议"全文，给出执行计划，列出会触达的外部系统。
2. **Dry-run**：能本地预演的步骤先空跑（生成草稿、打印命令而不执行），让用户确认输出无误。
3. **External writes 必须人工确认**：发邮件、发消息、发布内容、调用付费 API、修改远程仓库、转账或下单——执行前必须等用户明确授权。
4. **凭证占位符**：`YOUR_*` / `${VAR}` / `$VARIABLE` 由用户提供真实值；**永远不要**把凭证硬编码进配置文件。
5. **失败先报告**：报错时先停下汇报上下文，再尝试修复，不要默默重试或跳过安全检查（如 `--no-verify`）。
6. **Prompt 语言**：用例中的英文 prompt 通常效果最佳；中文版本（若有）见用例底部 "中国用户适配" 章节。
7. **不要修改本仓库案例**：除非用户明确要求贡献回上游，参考 [CONTRIBUTING.md](CONTRIBUTING.md)。

## Where to Look

| 文件 | 用途 |
|---|---|
| [INDEX.md](INDEX.md) | 49 个用例的扁平索引：路径 / 一句话摘要 / 风险标签 |
| [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md) | OpenClaw 术语 ↔ Hermes / Claude Code / Codex 等价物对照 |
| [AGENT-GUIDE.md](AGENT-GUIDE.md) | 用例文件结构与代码块执行规则的细则（人类与 agent 共用） |
| [CONTRIBUTING.md](CONTRIBUTING.md) | 贡献新用例的格式与收录标准 |
| `usecases/*.md` | 49 个用例本体，按文件名组织 |

## 推荐首批低风险用例

如果不知道从哪开始，从这些纯读取或仅本地写入的开始（详细风险标签见 [INDEX.md](INDEX.md)）：

1. [`daily-reddit-digest.md`](usecases/daily-reddit-digest.md) — 每日 Reddit 摘要 (`external-api`)
2. [`daily-youtube-digest.md`](usecases/daily-youtube-digest.md) — 关注频道每日新视频摘要 (`external-api`)
3. [`hf-papers-research-discovery.md`](usecases/hf-papers-research-discovery.md) — 每日 ML 论文筛选 (`external-api`)
4. [`second-brain.md`](usecases/second-brain.md) — 随手记的可搜索笔记库 (`writes-local`)
5. [`knowledge-base-rag.md`](usecases/knowledge-base-rag.md) — 个人 RAG 知识库 (`writes-local` + `external-api`)
6. [`semantic-memory-search.md`](usecases/semantic-memory-search.md) — 记忆向量搜索 (`writes-local`)
7. [`opik-openclaw-observability.md`](usecases/opik-openclaw-observability.md) — Opik 链路追踪与成本监控 (`external-api` + `writes-local`)

## Don'ts

- **不要** 把任何用例当成可直接对外发布的脚本——所有发布、外发、转账操作都属于 external write，必须先 dry-run 给用户确认。
- **不要** 假设 SOUL.md 是当前会话的强制 system prompt——它是 OpenClaw 的人格定义文件，其他 agent 不一定有等价物（见 `CONCEPT-MAPPING.md`）。
- **不要** 因为某 OpenClaw 术语找不到对应就跳过用例，先查 `CONCEPT-MAPPING.md` 找等价物。
- **不要** 主动修改 49 个用例本体或在它们里添加 frontmatter——它们是稳定的人类阅读资产。
- **不要** 在脚本和配置中硬编码 API Key / token / 个人凭证。
- **不要** 用本合集中的用例去自动化抓取或群发到平台——社交媒体类用例都附带平台风控提醒，请遵守。
- **不要** 编造 OpenClaw / Hermes / Claude / Codex 不存在的功能或命令；不确定就在回复中明确指出并请用户验证。

---

> 本文件是仓库 AI agent 入口的单源真相。`CLAUDE.md` 仅作指针。如需扩展，请优先精简或下放到 `INDEX.md` / `CONCEPT-MAPPING.md`，保持本文件少于 150 行。
