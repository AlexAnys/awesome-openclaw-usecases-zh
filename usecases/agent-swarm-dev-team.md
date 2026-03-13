# Agent Swarm 一人开发团队（全配置指南）

> 含国内适配：飞书/钉钉通知 / Codex 替代方案 / 国内监控平台

用 Codex 或 Claude Code 直接编码？它们只看到代码，看不到你的业务全貌。上下文窗口（context window）是零和博弈——填满代码就没有空间放业务上下文，填满客户历史就没有空间放代码。

OpenClaw 作为编排层改变了这个等式：它持有你所有的业务上下文（客户数据、会议记录、历史决策、成功与失败经验），并将这些上下文转化为精确的 Prompt 分发给各个编码 Agent。编码 Agent 专注代码，编排 Agent 把控战略——**通过上下文实现专业化，而非通过不同的模型**。

## 它能做什么

- **异构模型编排**：编排器（"Zoe"）根据任务类型自动选择 Codex（OpenAI 编码模型）或 Claude Code（Anthropic 编码工具），路由到最合适的模型
- **Git Worktree（工作树）隔离**：每个 Agent 在独立的 Worktree 和分支上工作，互不干扰
- **tmux（终端复用器）会话管理**：Agent 运行在 tmux 会话中，支持中途发送指令纠偏，无需终止重启
- **三模型代码审查**：每个 PR 自动经过 Codex、Gemini Code Assist、Claude Code 三重审查，互补盲区
- **Cron 监控循环**：每 10 分钟自动检测所有 Agent 状态（tmux 存活、PR 状态、CI 结果），失败自动重生（最多 3 次）
- **自改进 Prompt（提示词，Ralph Loop V2）**：失败时编排器结合业务上下文重写 Prompt，而非简单重试；成功模式自动沉淀为复用经验
- **主动发现工作**：早间扫描 Sentry 错误→自动修复、会后扫描笔记→生成功能需求、晚间更新日志和文档
- **完成才通知**：PR 通过全部检查（CI + 三模型审查 + 截图）后才推送 Telegram，避免信息噪音

## 所需技能

- OpenClaw 2026.2+ 版本
- `sessions_spawn` / `sessions_send`（多 Agent 协调）
- `telegram` 技能（通知渠道，国内可替换为飞书/钉钉）
- `file_read` / `file_write`（任务注册表管理）
- `cron`（定时监控）
- Git + [gh CLI](https://cli.github.com/)（PR 管理和 CI 状态检查）
- [tmux](https://github.com/tmux/tmux)（Agent 会话管理）
- Codex CLI 或 Claude Code CLI（编码 Agent）

## 核心架构

```
你（决策者）
 └── Zoe（OpenClaw 编排器，Obsidian Vault 中的业务上下文）
      ├── Codex Agent 1（feat/custom-templates，tmux: codex-templates）
      ├── Codex Agent 2（fix/billing-bug，tmux: codex-billing）
      ├── Claude Code Agent（feat/ui-refresh，tmux: cc-ui）
      └── Cron 监控脚本（每 10 分钟，check-agents.sh）
           ├── 检查 tmux 会话是否存活
           ├── 通过 gh CLI 检查 PR 和 CI 状态
           └── 失败 → Zoe 重写 Prompt → 重生 Agent
```

两层上下文分离是核心设计：

| | OpenClaw 编排器（Zoe） | 编码 Agent（Codex / Claude Code） |
|---|---|---|
| **上下文** | 客户数据、会议记录、历史决策、成功/失败模式 | 代码库、类型定义、测试文件、API 规范 |
| **职责** | 范围确定、Prompt 生成、Agent 调度、失败分析 | 编码、测试、提交、创建 PR |
| **权限** | Admin API、只读数据库、任务注册表 | 仅代码仓库和 CI |

## 8 步工作流

### Step 1：客户需求 → 与编排器确定范围

客户提出需求后，与编排器讨论。因为会议笔记自动同步到 Obsidian Vault，编排器已有完整上下文，无需额外解释。

编排器完成三件事：
1. 通过 Admin API 为客户充值或解锁功能（立即响应客户）
2. 通过只读数据库访问拉取客户现有配置（编码 Agent 永远没有生产数据库权限）
3. 生成包含完整上下文的 Prompt 并 Spawn 编码 Agent

### Step 2：生成 Agent

每个 Agent 获得独立的 Worktree 和 tmux 会话：

```bash
# 创建 worktree + 启动 agent
git worktree add ../feat-custom-templates -b feat/custom-templates origin/main
cd ../feat-custom-templates && pnpm install

tmux new-session -d -s "codex-templates" \
  -c "/path/to/worktrees/feat-custom-templates" \
  "$HOME/.clawdbot/run-agent.sh templates gpt-5.3-codex high"
```

启动编码 Agent 的命令：

```bash
# Codex
codex --model gpt-5.3-codex \
  -c "model_reasoning_effort=high" \
  --dangerously-bypass-approvals-and-sandbox \
  "Your prompt here"

# Claude Code
claude --model claude-opus-4.5 \
  --dangerously-skip-permissions \
  -p "Your prompt here"
```

tmux 的核心优势——中途纠偏，无需终止 Agent：

```bash
# Agent 方向错了？
tmux send-keys -t codex-templates "Stop. Focus on the API layer first, not the UI." Enter

# Agent 需要更多上下文？
tmux send-keys -t codex-templates "The schema is in src/types/template.ts. Use that." Enter
```

任务在 `.clawdbot/active-tasks.json` 中追踪：

```json
{
  "id": "feat-custom-templates",
  "tmuxSession": "codex-templates",
  "agent": "codex",
  "description": "Custom email templates for agency customer",
  "repo": "medialyst",
  "worktree": "feat-custom-templates",
  "branch": "feat/custom-templates",
  "startedAt": 1740268800000,
  "status": "running",
  "notifyOnComplete": true
}
```

### Step 3：Cron 监控循环

每 10 分钟运行 `.clawdbot/check-agents.sh`，100% 确定性逻辑、极低 Token 消耗：

- 检查 tmux 会话是否存活
- 通过 `gh` CLI 检查是否有已打开的 PR
- 检查 CI 状态
- CI 失败或关键审查意见 → 自动重生 Agent（最多 3 次）
- 只在需要人工介入时告警

不直接轮询 Agent（那会很贵），而是读取 JSON 注册表和外部状态。

### Step 4：Agent 创建 PR

Agent 通过 `gh pr create --fill` 提交 PR。此时**不通知你**——PR 创建不等于完成。

Definition of Done（确保你的 Agent 知道这个）：

- ✅ PR 已创建
- ✅ 分支已同步 main（无合并冲突）
- ✅ CI 通过（lint、类型检查、单元测试、E2E）
- ✅ Codex Review 通过
- ✅ Claude Code Review 通过
- ✅ Gemini Review 通过
- ✅ UI 变更包含截图

完成后任务注册表自动更新：

```json
{
  "status": "done",
  "pr": 341,
  "completedAt": 1740275400000,
  "checks": {
    "prCreated": true,
    "ciPassed": true,
    "claudeReviewPassed": true,
    "geminiReviewPassed": true
  },
  "note": "All checks passed. Ready to merge."
}
```

### Step 5：三模型自动代码审查

| 审查器 | 强项 | 说明 |
|--------|------|------|
| Codex | 边界情况、逻辑错误、竞态条件 | 最深入，误报率极低 |
| Gemini Code Assist | 安全漏洞、可扩展性问题 | 免费，会给出具体修复建议 |
| Claude Code | 验证其他审查器的发现 | 倾向过度保守，只关注 Critical 级别 |

三个审查器都直接在 PR 上发表评论。

### Step 6：自动化测试

CI 流水线包含：

- Lint + TypeScript 类型检查
- 单元测试
- E2E 测试
- Playwright 测试（对照与生产一致的预览环境）

规则：UI 变更的 PR 必须在描述中包含截图，否则 CI 直接失败。这大幅缩短了审查时间——无需手动点开预览环境。

### Step 7：人工审查

此时收到通知："PR #341 ready for review"。CI 已通过、三个 AI 审查器已批准、截图展示了 UI 变更。人工审查仅需 5-10 分钟，很多 PR 看完截图就可以直接合并。

### Step 8：合并与清理

PR 合并。每日 Cron 清理孤立的 Worktree 和过期任务注册表。

## Ralph Loop V2：自改进 Prompt 系统

传统 Ralph Loop 每个循环使用相同的 Prompt，积累的经验改善了检索质量，但 Prompt 本身是静态的。

本系统不同。Agent 失败时，编排器结合**业务上下文**分析原因并重写 Prompt：

- Agent 上下文溢出？→ "Focus only on these three files."
- Agent 方向偏了？→ "Stop. The customer wanted X, not Y. Here's what they said in the meeting."
- Agent 需要澄清？→ "Here's the customer's email and what their company does."

编排器还主动发现工作：

- **早间**：扫描 Sentry → 发现 4 个新错误 → 生成 4 个修复 Agent
- **会后**：扫描笔记 → 标记 3 个功能请求 → 生成 3 个 Codex Agent
- **晚间**：扫描 Git 日志 → 生成 Claude Code 更新 Changelog 和客户文档

奖励信号：CI 通过 + 三模型审查通过 + 人工合并。任何失败触发循环。成功模式被记录：

- "This prompt structure works for billing features."
- "Codex needs the type definitions upfront."
- "Always include the test file paths."

## 模型选择指南

| 模型 | 适用场景 | 特点 |
|------|----------|------|
| Codex (gpt-5.3-codex) | 后端逻辑、复杂 Bug、多文件重构 | 慢但深入，用于 90% 的任务 |
| Claude Code (claude-opus-4.5) | 前端、Git 操作 | 快速，权限问题更少 |
| Gemini | UI 设计规范 | 先用 Gemini 生成 HTML/CSS 规范，再交给 Claude Code 实现 |

编排器根据任务类型自动路由：计费系统 Bug → Codex、按钮样式修改 → Claude Code、新仪表板设计 → Gemini 出稿 + Claude Code 实现。

## 如何设置

将本文内容复制给 OpenClaw，使用以下提示词：

```text
Implement this agent swarm setup for my codebase:

1. Create .clawdbot/ directory with:
   - run-agent.sh: Script to spawn coding agents in tmux sessions with worktree isolation
   - check-agents.sh: Deterministic monitoring script that checks tmux sessions, PR status (via gh cli), and CI status
   - active-tasks.json: Task registry tracking all running agents

2. Set up cron job to run check-agents.sh every 10 minutes

3. Configure notification channel (Telegram/Feishu/DingTalk) for "definition of done" alerts only

4. Read my codebase structure and create agent spawn templates appropriate for my tech stack
```

OpenClaw 会分析代码库、创建脚本、配置 Cron 监控。约 10 分钟完成初始配置。

## 实用建议

- **从小开始**：先用 1-2 个 Agent 跑通全流程（Spawn → 编码 → PR → 审查 → 合并），确认一切正常后再扩展
- **RAM 是真正瓶颈**：每个 Agent 需要独立的 `node_modules` 和构建进程。16GB 内存最多同时跑 4-5 个 Agent，且不能同时触发构建。需要大规模并行建议 64GB+ 内存
- **成本参考**：Claude ~$100/月 + Codex ~$90/月，入门可以从 $20/月 起步
- **Definition of Done 是关键**：Agent 必须知道"PR 创建≠完成"，需包含 CI 通过、审查通过、截图（如适用）
- **tmux 优于 `-p` 模式**：tmux 支持中途发送指令纠偏，不需要杀掉重启 Agent
- **不要直接轮询 Agent**：用确定性脚本检查 tmux / PR / CI 外部状态，极低 Token 消耗
- **UI 截图规则**：强制 PR 包含截图可以大幅缩短审查时间——看截图比点开预览环境快得多

## 实际效果

来自作者 4 周的生产环境数据（用于真实 B2B SaaS 产品）：

- 最高单日 94 commits（当天有 3 个客户会议，未打开编辑器）
- 日均约 50 commits
- 30 分钟内完成 7 个 PR（从想法到生产）
- 中小任务一次通过率极高，几乎不需要人工干预
- 功能请求当天交付 → 提升 leads 到付费客户的转化率

## 相关链接

- [原文 (X/Twitter)](https://x.com/elvissun/status/2025920521871716562) — Elvis Sun 的完整文章
- [中文版 (知乎)](https://zhuanlan.zhihu.com/p/2010127051726792220) — 含完整图表
- [OpenClaw 官方仓库](https://github.com/openclaw/openclaw)
- [Anthropic：构建有效的智能体](https://www.anthropic.com/research/building-effective-agents)

## 中国用户适配

### 通知渠道替代

| 原方案 | 国内替代 | 说明 |
|--------|----------|------|
| Telegram | 飞书自定义机器人 | Webhook 推送，`curl` 即可调用，参考 [飞书 AI 助手](cn-feishu-ai-assistant.md) |
| Telegram | 钉钉自定义机器人 | 支持 Webhook + 加签验证，参考 [钉钉 AI 助手](cn-dingtalk-ai-assistant.md) |
| Telegram | 企业微信应用消息 | 通过应用 API 推送，参考 [企业微信 AI 助手](cn-wecom-ai-assistant.md) |

### 监控与错误追踪替代

| 原方案 | 国内替代 | 说明 |
|--------|----------|------|
| Sentry | 阿里云 ARMS | 应用实时监控，支持自动告警触发 Agent |
| Sentry | 腾讯云前端性能监控 | 错误上报和性能追踪 |

### 代码托管与 CI

- **GitHub + gh CLI**（推荐保留）：`gh` CLI 在国内可用，可能需要代理加速
- **如使用 Gitee**：`check-agents.sh` 中的 `gh` 命令需替换为 Gitee API 调用，CI 切换到 Jenkins 等

### Obsidian Vault

Obsidian 在国内可正常使用，无需替代。如团队使用语雀或飞书文档，可通过 API 同步会议记录到本地文件系统供编排器读取。

### Codex 可用性

Codex CLI 需要 OpenAI API 访问，国内需要科学上网。如无法使用，可全部改用 Claude Code：

```text
# 将所有任务改为 Claude Code 执行
claude --model claude-opus-4.5 \
  --dangerously-skip-permissions \
  -p "Your prompt here"
```

模型选择指南中 Codex 的任务可分配给 `claude-opus-4.5`（深度推理）或 `claude-sonnet-4.5`（速度优先）。

### 国内适配飞书通知提示词

以下提示词将 Telegram 通知替换为飞书：

```text
## Notification Setup (Feishu Webhook)

When a PR passes ALL checks (CI green + 3 code reviews approved + screenshots if UI changes):
1. POST to Feishu webhook: $FEISHU_WEBHOOK_URL
2. Message payload:
   {
     "msg_type": "interactive",
     "card": {
       "header": {"title": {"content": "PR #{number} Ready for Review", "tag": "plain_text"}},
       "elements": [{"tag": "div", "text": {"content": "**{title}**\nBranch: {branch}\nCI: ✅ Reviews: ✅", "tag": "lark_md"}}]
     }
   }
3. ONLY notify on "definition of done" — never on intermediate states
```

---

**灵感来源**：[Elvis Sun](https://x.com/elvissun/status/2025920521871716562) — 独立创始人，用 OpenClaw 编排 Codex + Claude Code Agent 舰队构建 B2B SaaS 产品（Agentic PR），日均 50+ commits，一人完成开发团队的工作量。他将此描述为"从管理 Claude Code，到管理一个管理 Claude Code 和 Codex 舰队的 OpenClaw Agent"。

**原文链接**：[Elvis Sun on X](https://x.com/elvissun/status/2025920521871716562) · [知乎中文版](https://zhuanlan.zhihu.com/p/2010127051726792220)
