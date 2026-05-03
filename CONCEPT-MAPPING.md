# CONCEPT-MAPPING.md

把 OpenClaw 的核心概念**近似映射**到其他主流个人 AI 智能体生态。本仓库 49 个用例都用 OpenClaw 术语写就，遇到不熟悉的术语回到这张表查找等价物即可，**无需改写用例本体**。

> **这是一份近似映射，不是官方兼容矩阵。** 各家 agent 的语义、加载机制、目录约定都在演化，使用前请以**目标 agent 的官方文档与本地实测**为准。
>
> 标注 `—` 表示**无原生等价物**，需在该 agent 中手动模拟（通常是把行为写进 system prompt 或外部自动化）。
> 标注 ⚠️ 表示**语义近似而非完全对等**，请阅读"说明"列了解差异。

## 核心概念对照表

| OpenClaw 概念 | Hermes (Nous Research) | Claude Code (Anthropic) | Codex (OpenAI) | 说明 |
|---|---|---|---|---|
| **Skill**（教智能体做某事的知识包） | `~/.hermes/skills/<cat>/<skill>/SKILL.md` | `.claude/skills/`（项目）/ `~/.claude/skills/`（用户） | `~/.agents/skills/`（个人）/ `<repo>/.agents/skills/`（团队） | OpenClaw / Hermes 使用 agentskills.io 风格的 `SKILL.md`；其余 agent 的技能格式相近但触发条件与安装命令以官方文档为准。 |
| **Cron Job / Heartbeat**（按时间表执行 / 定期巡检） | Hermes cron + closed learning loop | ⚠️ 无原生 cron；用系统 cron / launchd / GitHub Actions 触发 `claude` CLI | Codex automations（Web / Cloud 计划任务） | 跨 agent 落地通常退化到**系统 cron / launchd / GitHub Actions** + 进程入口 prompt。 |
| **Channel**（连接 Telegram / 飞书 / Discord / 钉钉等 IM 平台） | Hermes gateway（IM 适配层） | — 无原生 channel | — 无原生 channel | Claude Code / Codex 没有"自动监听 IM 平台并触发 agent"的内建能力；Channel 步骤需用 webhook + 中间件（n8n、自建 server）替代，或保留 OpenClaw 端处理 IM 入口。 |
| **Memory**（`MEMORY.md` / `USER.md` / 持久化记忆） | Honcho / Hermes memory store | Claude Code auto-memory（`~/.claude/projects/.../memory/`） | ⚠️ Codex 无统一持久 memory；可放进 AGENTS.md 或 `.agents/memory/` 自管 | OpenClaw 的 `MEMORY.md` 是文件型记忆，跨 agent 可作为 plain markdown 复用，但加载机制各家不同。 |
| **SOUL.md**（人格、语气、边界） | Hermes user model + system prompt | Claude Code 系统提示 / `CLAUDE.md` 内 persona 段 | Codex `AGENTS.md` 的 persona 段 | ⚠️ 无强制等价物。建议把 SOUL.md 内容粘到对应 agent 的 system prompt 或 AGENTS.md 顶部，作为 persona 定义。 |
| **AGENTS.md**（智能体操作手册） | 会读取启动目录下的 `AGENTS.md`；建议从仓库根启动以保证根上下文加载 | ⚠️ Claude Code 截至 2026-05 本地实测不自动读取；本仓库通过 `CLAUDE.md → @AGENTS.md` 指针兼容 | 自动读取（CLI / Web / Cloud） | 跨工具事实标准之一；Hermes 不做 Codex 那种目录树多层合并。 |
| **Sub-agent / Delegate**（派分身并行处理） | `delegate_task` / 子 Hermes 进程 | Claude Code subagents（`.claude/agents/*.md`） | Codex MCP server 调用其他 agent | 编排粒度差异较大；OpenClaw 的"子智能体"在 Claude Code 通常对应 subagents.md，在 Codex 通常通过 MCP delegation。 |
| **Workspace**（智能体的工作目录） | Hermes home（`~/.hermes/`） | 当前 git repo + `~/.claude/` | 当前 git repo + `~/.codex/` + `~/.agents/` | 用例中提到的 Workspace 路径需按目标 agent 调整。 |
| **MCP**（Model Context Protocol） | Hermes MCP（兼容） | Claude Code MCP（原生） | Codex MCP（原生） | 三家都支持 MCP，OpenClaw 用例中提到的 MCP server 通常可直接复用。 |
| **Browser / Web Tools**（浏览器抓取、HTTP 请求） | Hermes built-in tools + MCP browser | Claude Code WebFetch / Bash + curl / Playwright MCP | Codex web tools / Playwright MCP | 工具命名各异但语义可对应；"打开浏览器抓取 X"在任何 agent 中都能完成。 |
| **File Tools**（读写本地文件） | Hermes file tools | Read / Write / Edit / Glob / Grep | Codex read / write / exec | 完全对应，无需翻译。 |
| **Prompt**（你给 agent 的指令） | 同 | 同 | 同 | 用例中的英文 prompt 在所有 agent 中通常效果最佳；中文 prompt 见用例底部"中国用户适配"段。 |
| **Node**（手机 / 平板等"分布式节点"） | — | — | — | OpenClaw 独有概念。其他 agent 用 IM bot / Web UI / 远程 SSH 替代。 |

## 翻译速查（执行用例时）

读到 OpenClaw 用例中的某个步骤时，按下表替换：

- 「安装这个 Skill」→ 在你的 agent 对应 skills 目录里放好 SKILL.md，或用 agent 自己的 install 命令。
- 「配置 Channel（连飞书 / 钉钉 / Discord）」→ Claude Code / Codex 改用 webhook + 系统 cron 触发；保留 OpenClaw 实例处理 IM 入口是最省事的路径。
- 「设置 Cron / Heartbeat」→ 改用系统 cron / launchd / GitHub Actions 调用 agent 入口。
- 「子智能体处理 X」→ Claude Code 的 `subagents.md` / Codex 的 MCP delegation / Hermes 的 `delegate_task`。
- 「写入 MEMORY.md」→ 写进对应 agent 的 memory 路径（见上表 Memory 行）。
- 「按 SOUL.md 行事」→ 把 SOUL.md 内容粘到 system prompt 或 AGENTS.md 顶部 persona 段。

## 不确定 / 边界

诚实标注以下空洞，避免下游用户被误导：

- **Hermes 会读取启动目录下的 `AGENTS.md`，但不会沿目录树合并多份 AGENTS.md（Codex 行为）**。如果 Hermes 不在仓库根目录运行，落地路径：
  1. 会话中粘贴用例 raw URL 或文件路径，让 Hermes 拉取并解析；
  2. 把常用用例整理为 Hermes `SKILL.md` 后再安装；
  3. 是否存在一键导入 OpenClaw 配置的命令请以 Hermes 官方文档为准（本仓库不假定该命令存在）。
- **Channel** 在 Claude Code / Codex 中没有干净的等价物。建议保留 OpenClaw 实例处理 IM 入口，把分析或写作子任务交给其他 agent。
- **Codex 持久 Memory** 没有统一标准；如用例依赖跨会话记忆，需把状态写进仓库内的 markdown 或外部数据库。
- **SKILL.md 兼容性**：OpenClaw / Hermes 使用 agentskills.io 风格；Claude Code / Codex 的技能格式相近但触发条件与安装方式不完全一致，跨生态复用时需测试。
