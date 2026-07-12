# CONCEPT-MAPPING.md

把 OpenClaw 的核心概念映射到其他主流个人 AI 智能体生态。本仓库的 50 个用例都用 OpenClaw 术语写就，遇到不熟悉的术语时回到这张表查找等价物即可，**无需改写用例本体**。

> 这些事实经过 2026-06 的官方文档核实、2026-07 抽样复核（含 `hermes claw migrate` 命令实测存在）；标注了置信边界。各 agent 行为更新较快，执行前以其官方文档为准。

> 标注 `—` 表示**无原生等价物**，需在该 agent 中手动模拟（通常是把行为写进 system prompt 或外部自动化）。
> 标注 ⚠️ 表示**语义近似而非完全对等**，请阅读"说明"列了解差异。

## 核心概念对照表

| OpenClaw 概念 | Hermes (Nous Research) | Claude Code (Anthropic) | Codex / GPT (OpenAI) | 说明 |
|---|---|---|---|---|
| **Skill**（教智能体做某事的知识包） | `~/.hermes/skills/<cat>/<skill>/SKILL.md` | `.claude/skills/`（项目）/ `~/.claude/skills/`（用户） | `~/.agents/skills/`（个人）/ `<repo>/.agents/skills/`（团队） | OpenClaw / Hermes 使用 agentskills.io 风格的 `SKILL.md`；Claude Code / Codex 的技能格式相近但触发条件和安装命令需以各自文档为准。 |
| **Cron Job / Heartbeat**（按时间表执行 / 定期巡检并主动汇报） | Hermes 内置 cron 调度器（可投递到任意平台）+ closed learning loop | ⚠️ 已有原生调度：`/schedule` 创建在 Anthropic 云端运行的 "routines"（预设周期 / 一次性 / 自定义 cron，最小间隔 1 小时，research preview，需付费订阅账号），另有 Desktop scheduled tasks 与 in-session `/loop`；系统 cron / launchd / GitHub Actions 作为自托管/CI 备选 | Codex automations（Web / Cloud 计划任务） | 各家都已有某种定时能力，但语义不同（如 Claude Code routines 最小 1 小时、云端运行，不同于 OpenClaw 的 Heartbeat 主动巡检）；自托管场景仍可退化到系统 cron / launchd / GitHub Actions + 进程入口 prompt。 |
| **Channel**（连接 Telegram / 飞书 / Discord / 钉钉等 IM 平台） | Hermes gateway（IM 适配层） | — 无原生 channel | — 无原生 channel | Claude Code / Codex 没有"自动监听 IM 平台并触发 agent"的内建能力；用例中的 Channel 步骤需用 webhook + 中间件（如 n8n、自建 server）替代，或保留 OpenClaw 端处理 IM 入口。 |
| **Memory**（`MEMORY.md` / `USER.md` / 持久化记忆） | 内置 recall = FTS5 会话检索 + agent 维护的 `MEMORY.md` / `USER.md`；Honcho / mem0 / supermemory 等为**可选**外部 provider | Claude Code auto-memory（默认开启，需 v2.1.59+；`~/.claude/projects/<project>/memory/`，含 `MEMORY.md` 入口 + 话题文件） | ⚠️ Codex 无统一持久 memory；可放进 AGENTS.md 或 `.agents/memory/` 自管 | OpenClaw 的 `MEMORY.md` 是文件型记忆，跨 agent 可作为 plain markdown 复用，但加载机制各家不同。 |
| **SOUL.md**（人格、语气、边界） | Hermes user model + system prompt | Claude Code 系统提示 / `CLAUDE.md` 内 persona 段 | Codex `AGENTS.md` 的 persona 段 | ⚠️ 无强制等价物。建议把 SOUL.md 内容粘到对应 agent 的 system prompt 或 AGENTS.md 顶部，作为一段 persona 定义。 |
| **AGENTS.md**（智能体操作手册） | 会话开始读取工作目录下 `AGENTS.md`（优先级 `.hermes.md → AGENTS.md → CLAUDE.md → .cursorrules`） | ⚠️ 按官方 memory 文档，Claude Code 读 `CLAUDE.md` 而**不自动读 AGENTS.md**；本仓库用 `CLAUDE.md` 内 `@AGENTS.md` 导入兼容（也可 `ln -s AGENTS.md CLAUDE.md`） | 开始工作前自动读取，并沿目录树从根向下**合并**（最近的覆盖更靠前的，上限 32 KiB） | AGENTS.md 是 2025 年起源自 Codex、2025-12 进入 Linux Foundation Agentic AI Foundation 的跨工具开放约定（60,000+ 项目采用）；但各家行为不同：Codex 合并目录树、Hermes 只读 CWD 不做多层合并、Claude Code 走 CLAUDE.md 导入。建议在仓库根运行。 |
| **Sub-agent / Delegate** ⚠️（派分身并行处理） | `delegate_task` / 子 Hermes 进程 | Claude Code subagents（`.claude/agents/*.md`，带 YAML frontmatter） | Codex MCP server 调用其他 agent | ⚠️ 编排粒度差异较大：OpenClaw 的"子智能体"在 Claude Code 对应 `.claude/agents/*.md`，在 Codex 通常通过 MCP delegation，在 Hermes 是 `delegate_task`。 |
| **Workspace**（智能体的工作目录） | Hermes home（`~/.hermes/`） | 当前 git repo + `~/.claude/` | 当前 git repo + `~/.codex/` + `~/.agents/` | 用例中提到的 Workspace 路径需按目标 agent 调整。 |
| **MCP**（Model Context Protocol，工具连接） | Hermes MCP（原生，client 为可选依赖，未装 `mcp` 包则静默禁用） | Claude Code MCP（原生） | Codex MCP（原生，server 经实验性 `codex mcp-server`） | 四家（含 OpenClaw 自身）均原生支持 MCP，用例中提到的 MCP server 通常可直接复用。注：OpenClaw 设计上更鼓励 CLI/shell 工具而非 MCP schema 注入以省 token。 |
| **Browser / Web Tools**（浏览器抓取、HTTP 请求） | Hermes built-in tools + MCP browser server | Claude Code WebFetch / Bash + curl / Playwright MCP | Codex web tools / Playwright MCP | 工具命名各异但语义可对应；用例中"打开浏览器抓取 X"在任何 agent 中都能完成。 |
| **File Tools**（读写本地文件、生成报告） | Hermes file tools | Read / Write / Edit / Glob / Grep | Codex read / write / exec | 完全对应，无需翻译。 |
| **Prompt**（你给 agent 的指令） | 同 | 同 | 同 | 用例中的英文 prompt 在所有 agent 中通常效果最佳；中文 prompt 见用例底部"中国用户适配"段。 |
| **Node**（手机 / 平板等"分布式节点"） | — | — | — | OpenClaw 独有概念。其他 agent 用 IM bot / Web UI / 远程 SSH 替代。 |

## 翻译速查（执行用例时）

读到 OpenClaw 用例中的某个步骤时，按下表替换：

- 「安装这个 Skill」→ 在你的 agent 对应 skills 目录里放好 SKILL.md，或用 agent 自己的 install 命令。
- 「配置 Channel（连飞书 / 钉钉 / Discord）」→ Claude Code / Codex 改用 webhook + 系统 cron 触发；保留 OpenClaw 实例处理 IM 入口是最省事的路径。
- 「设置 Cron / Heartbeat」→ 优先用 agent 自带的调度（Claude Code `/schedule` routines、Hermes 内置 cron、Codex automations）；自托管/CI 场景退化到系统 cron / launchd / GitHub Actions 调用 agent 入口。
- 「子智能体处理 X」→ Claude Code 的 `.claude/agents/*.md` / Codex 的 MCP delegation / Hermes 的 `delegate_task`。
- 「写入 MEMORY.md」→ 写进对应 agent 的 memory 路径（见上表 Memory 行）。
- 「按 SOUL.md 行事」→ 把 SOUL.md 内容粘到 system prompt 或 AGENTS.md 顶部 persona 段。

## 不确定 / 边界

诚实标注以下空洞，避免下游用户被误导：

- **Hermes 会读取当前工作目录下的 `AGENTS.md`，但不会像 Codex 一样沿目录树合并多份 AGENTS.md**。如果 Hermes 没有在本仓库根目录运行，则需要：
  1. 会话中粘贴用例 raw URL 或文件路径，让 Hermes 拉取并解析；
  2. 将常用用例整理成 Hermes `SKILL.md` 后再安装；
  3. 使用 `hermes claw migrate` 导入已有 OpenClaw 配置。
- **OpenClaw 的 Channel** 在 Claude Code / Codex 中没有干净的等价物。建议保留 OpenClaw 实例处理 IM 入口，把分析或写作子任务交给其他 agent。
- **Codex 的持久 Memory** 没有统一标准；如果用例依赖跨会话记忆，需把状态写进仓库内的 markdown 或外部数据库。
- **SKILL.md 兼容性**：`SKILL.md` 是 agentskills.io 的开放标准（最初由 Anthropic 开源），OpenClaw / Hermes / Claude Code / Codex 均声明采用。基础 skill（`name` + `description` + 正文）可跨工具直接移植；但**并非 100% 一致**——各家有私有 frontmatter，发现目录也不同（Claude Code 用 `.claude/skills/`，Codex 与 OpenClaw 都用 `.agents/skills/`），且 `name` 在 Claude Code 可选、在 spec 里必填。跨生态复用时需测试。
