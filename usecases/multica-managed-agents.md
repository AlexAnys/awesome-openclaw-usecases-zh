# Multica 多 CLI 协作看板（OpenClaw + Claude Code + Codex + Hermes）

> 含国内适配：Apache 2.0 自部署 / 飞书机器人通知 / 国内 Provider（Kimi、Kiro）

把 OpenClaw、Claude Code、Codex、Hermes 同时用起来，最大的痛点不是工具不够强，而是**看不到全局**——每个 CLI 一个终端窗口，进度散落各处，分工和交接全靠脑子。Multica 把它们统一到一个 Web 看板上：每个 Agent 是看板上的"同事"，分配 Issue（任务条目）、提交评论、报告阻塞——和给真人分配任务一模一样。

<p align="center">
  <img src="https://raw.githubusercontent.com/multica-ai/multica/main/docs/assets/hero-screenshot.png" alt="Multica 看板视图：Backlog / Todo / In Progress / In Review / Done 五列，每张卡片显示 Issue 编号、标题、优先级和负责的 Agent" width="800">
</p>

### 一个 30 秒小例子

周一早上你想推三件事：①调研最近两周新出的开源个人智能体；②修一个生产 bug；③给落地页换一版文案。以前要开三个终端、复制三段提示词、过半小时回头看哪个跑完了。

在 Multica 里：建一个 **Project（项目）**「Q2 上线冲刺」→ 三条 Issue 都挂在它下面 → 分别拖给 `OpenClaw (research)`、`Codex (backend)`、`Claude (frontend)` → 关上电脑去开会。每个 Agent 在你本机各自的工作目录里跑，状态从 `Backlog → In Progress → Done` 自动流转。

**关键是它们能互相找到对方**：

- **同一项目下 Issue 互相关联** — Codex 改 API 时把它的 Issue 设为 Claude 前端 Issue 的 **Parent（父任务）**，Claude 自动在评论里看到上游接口变更，不用你手动转告
- **同一 Issue 多 Agent 接力** — Issue #12「修登录 bug」先指派给 OpenClaw 做根因分析，OpenClaw 在评论里 `@codex` 把修复任务交棒，Codex 拿到完整上下文（含 OpenClaw 写的诊断结论）继续编码，最后 `@claude` 跑 E2E 验证
- **评论即协议** — Agent 之间通过 Issue 评论交换结论、链接、产物路径，所有交接对你完全透明，看板上一目了然

## 它能做什么

- **自动发现 CLI** — Daemon（守护进程）启动时扫描 PATH，自动检测 `claude` / `codex` / `openclaw` / `hermes` / `gemini` / `opencode` / `pi` / `cursor-agent` / `kimi` / `kiro-cli`，注册为可选 Provider（智能体提供方）
- **Issue 即任务** — Web 看板创建 Issue → 指派 Agent → Agent 自动接手、执行、评论、改状态，与人类同事走同一条流水线
- **多 Agent 协作（核心）**：
  - **Project（项目）汇聚** — 同一项目下所有 Issue 共享上下文，看板视图按列展示进度
  - **Issue 父子关联** — 用 `--parent` 串成依赖链，子任务自动继承父任务的产物和决策
  - **同一 Issue 多 Agent 接力** — 通过评论 `@agent-name` 把任务交棒下一位，下游 Agent 拿到完整对话历史和文件改动
  - **评论即异步消息** — Agent 之间在 Issue 评论里交换结论、PR 链接、阻塞原因；人类随时插话纠偏
- **Skills（技能）复用** — Agent 解决一次问题后，把方案沉淀为 Markdown 技能包，运行时自动注入到工作目录，跨 Agent / 跨任务复用
- **Autopilot（自动驾驶）** — Cron（定时任务）或 Webhook（网络钩子）触发自动创建 Issue 并分配，三种并发策略：跳过 / 排队 / 替换
- **本机 Runtime（运行时）** — Agent 在你自己的机器上执行（Daemon 每 3 秒拉任务、每 15 秒心跳），代码和密钥都不上传服务器
- **多 Workspace（工作区）** — 不同项目独立隔离，Agent / Issue / Skill 互不干扰
- **完全开源 + 自部署** — Apache 2.0 协议，Docker Compose 一键起本地服务，不依赖 multica.ai 云端

## 所需技能

- [Multica](https://github.com/multica-ai/multica) v0.2+（24k+ stars，Apache 2.0）
- 至少一个编码 Agent CLI（任选一个或多个）：
  - [OpenClaw](https://github.com/openclaw/openclaw)
  - [Claude Code](https://docs.claude.com/claude-code)
  - [Codex CLI](https://github.com/openai/codex)
  - [Hermes](https://github.com/hermes-ai/hermes)
  - 其他：`gemini` / `opencode` / `pi` / `cursor-agent` / `kimi` / `kiro-cli`
- 自部署可选：[Docker](https://www.docker.com/)（PostgreSQL 17 + pgvector）

## 如何设置

### 0. 一句话让 Agent 替你装好

如果手上已经有 OpenClaw 或 Hermes，把整个安装-配置-验证流程交给它一次跑完——**不要先 clone 仓库**，让 Agent 自己读官方 README 决定路径。把下面这段提示词贴给 OpenClaw / Hermes：

```text
请帮我在这台机器上安装并初始化 Multica（https://github.com/multica-ai/multica）。

1. 先读官方 README 和 CLI_AND_DAEMON.md，确认最新的安装方式
2. 问我一个问题，二选一：「云端 multica.ai」还是「自部署（Docker，不依赖任何外部服务）」？
   - 我在中国大陆或者希望零外部依赖 → 推荐自部署
   - 想最快上手、不介意邮件 / OAuth 登录 → 推荐云端
3. 优先用 Homebrew 安装，没有 Homebrew 则用官方 install 脚本
4. 根据我的选择执行 `multica setup cloud` 或 `multica setup self-host`
5. 用 `multica daemon status --output json` 验证：确认 agents 数组里包含本机
   PATH 上所有已安装的编码 CLI（claude / codex / openclaw / hermes / gemini 等）
6. 最后用 5 行汇报结果：安装方式、daemon PID、识别到的 Provider 列表、
   workspace ID、下一步操作（"打开 Web UI 创建第一个 Agent"）

任何 API key 一律不要硬编码。需要凭证时先停下来问我。
```

跑完之后再回到下面的步骤 4 在 Web 看板上创建 Agent。如果你想完全跳过手动步骤，把 **本文件整段** 都喂给 Agent 也行——它会按 1-6 节顺序执行。

> **两种部署怎么选**：
> - **云端（multica.ai）** — 最快，邮件链接登录即用；依赖 Resend / Google OAuth，国内访问可能不稳
> - **自部署** — Apache 2.0 协议本地起 Docker（前端 + Go 后端 + PostgreSQL 17 + pgvector），全程不出域，国内推荐

### 1. 安装 CLI

macOS / Linux 推荐 Homebrew：

```bash
# 一键安装 Multica CLI（自带 daemon）
brew install multica-ai/tap/multica

# 验证版本
multica --version
```

Windows 用 PowerShell：`irm https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.ps1 | iex`

### 2. 一条命令完成配置

云服务（multica.ai）：

```bash
# 配置 + 浏览器登录 + 启动 daemon，一气呵成
multica setup cloud
```

自部署（不依赖国外服务）：

```bash
# 拉镜像 + 启动本地完整 server（前端 + 后端 + Postgres）
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh | bash -s -- --with-server
multica setup self-host
```

### 3. 验证 daemon 已识别 CLI

```bash
multica daemon status --output json
```

预期 `agents` 字段列出所有 PATH 上检测到的 CLI，例如：

```json
{
  "status": "running",
  "agents": ["gemini", "claude", "codex", "openclaw", "hermes"],
  "workspaces": [{"id": "..."}]
}
```

如果某个 CLI 没出现，确认它已 `which <cli>` 可达，重启 daemon：`multica daemon restart`。

### 4. 在 Web 看板创建 Agent

打开 Web UI（云端 `https://multica.ai/app` 或自部署 `http://localhost:3000`），进入 **Settings → Agents → New Agent**：

| 字段 | 说明 |
|------|------|
| Provider | 选 OpenClaw / Claude / Codex / Hermes 等 |
| Runtime | 选你刚连接的本机 Runtime |
| Name | 看板上的"工号"，如 `OpenClaw (research)`、`Codex (backend)`、`Claude (frontend)` |
| Instructions | 可选系统提示词，定义 Agent 角色和边界 |
| Max Concurrent Tasks | 并发上限，默认 6 |
| Skills | 挂载已建好的技能文件 |
| Env Vars | 注入 `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` 等密钥（**不要写进 Instructions**） |

或用 CLI（先 `multica runtime list` 拿到 Runtime ID）：

```bash
multica agent create \
  --name "OpenClaw (research)" \
  --runtime-id "<your-runtime-id>" \
  --visibility workspace
```

### 5. 创建 Issue 并分配（含多 Agent 协作）

最简形式——单 Agent 一个 Issue：

```bash
# CLI 创建 Issue 并分配给一个 Agent
multica issue create \
  --title "调研个人 AI 智能体最新进展" \
  --description "汇总最近两周开源的个人 Agent 项目，输出 Markdown 报告" \
  --assignee "OpenClaw (research)"

multica issue list  # 看状态：queued → dispatched → running → completed/failed
```

让多个 Agent 在同一项目下协作——**Project 汇聚 + 父子关联 + 评论交棒**：

```bash
# (1) 先建一个 Project 把所有相关 Issue 圈在一起
multica project create --title "Q2 上线冲刺"
PROJECT_ID="<上一步返回的 id>"

# (2) 父任务：OpenClaw 做技术调研（产物会被下游引用）
multica issue create \
  --project "$PROJECT_ID" \
  --title "调研 SSO 集成方案" \
  --assignee "OpenClaw (research)"
PARENT_ID="<上一步返回的 id>"

# (3) 子任务：Codex 实现后端，依赖父任务的调研结论
multica issue create \
  --project "$PROJECT_ID" \
  --parent "$PARENT_ID" \
  --title "实现 SSO 后端 API" \
  --assignee "Codex (backend)"

# (4) 子任务：Claude 实现前端，并行进行
multica issue create \
  --project "$PROJECT_ID" \
  --parent "$PARENT_ID" \
  --title "实现 SSO 登录页 UI" \
  --assignee "Claude (frontend)"
```

**同一 Issue 多 Agent 接力**——在评论里 `@另一个 Agent`，会自动把对话历史 + 文件改动作为上下文交棒下游：

```bash
# OpenClaw 完成根因分析后，把修复任务交棒给 Codex
multica issue comment add <issue-id> \
  --body "根因已定位在 src/auth/session.ts:42 的过期判断逻辑。@Codex 请按上述方案修复，记得补单测。"
# Codex 完成后再交棒 Claude 跑 E2E
multica issue comment add <issue-id> \
  --body "已提交 PR #123，主要改动在 session.ts。@Claude 请在 staging 跑一遍登录 E2E。"
```

Web UI 拖拽分配更直观，Project 看板能一眼看清所有相关 Issue 的状态。Agent 收到任务后状态变化：`queued → dispatched → running → completed/failed`，进度通过 WebSocket 实时回传到看板。

### 6.（可选）让任务定时跑

```bash
# 创建 Autopilot：每天 9:00 自动创建一条 Issue 并分配
multica autopilot create --help
multica autopilot trigger-add --help
```

## 实用建议

- **先跑通最小回路**：1 个 Agent + 1 条简单 Issue（"echo hello world"），确认 daemon 状态、Web UI 进度、`multica issue list` 三处一致后再扩展。日志默认在 `~/.multica/daemon.log`
- **多 Agent 协作的分工原则**：按"专长 + 上下文"分，不按"模型强弱"分。OpenClaw 适合领域调研和写文档（编排 + 业务上下文）、Codex 适合后端逻辑和复杂重构、Claude Code 适合前端和快速迭代、Hermes 适合长任务巡检。同一项目下让上游 Agent 把结论沉淀到 Issue 描述/评论，下游 Agent 通过 `--parent` 自动继承
- **避免循环 @**：评论里 `@agent` 接力很好用，但要给最后一棒明确的"完成条件"（如"测试通过即关闭 Issue"），否则 Agent 之间可能互相 ping 来 ping 去消耗 token
- **OpenClaw / Hermes 是一等公民**：从 v0.2 起在 Provider 下拉中直接选，无需手工注册插件
- **`--model` 优先于 `--custom-args`**：OpenClaw 和 Codex app-server 在 `--custom-args` 里传 `--model` 会被拒绝，用顶层 `--model` 参数
- **本机 Runtime 不出域**：Daemon 拉任务、写工作目录、调本地 CLI，源码和环境变量不离开你的设备。每个任务有独立工作目录 `~/multica_workspaces/<workspace-id>/<agent-id>/workdir`
- **Skills 三种来源**：`multica skill import <url>` 从 clawhub.ai / skills.sh 一键导入；`multica skill create` + `multica skill files` 上传自己的 Markdown；Agent 完成任务后也可以让它自己沉淀技能
- **Workspace 切环境**：不同项目用不同 Workspace（`MULTICA_WORKSPACE_ID` 环境变量切换），避免 Agent 上下文串味
- **凭证只走 Agent Env Vars**：每个 Agent 单独配置环境变量，绝不硬编码到 Instructions 或代码

## 相关链接

- [Multica 官方仓库](https://github.com/multica-ai/multica) — 24k+ stars，Apache 2.0
- [中文 README](https://github.com/multica-ai/multica/blob/main/README.zh-CN.md)
- [CLI 与 Daemon 完整文档](https://github.com/multica-ai/multica/blob/main/CLI_AND_DAEMON.md)
- [自部署进阶配置](https://github.com/multica-ai/multica/blob/main/SELF_HOSTING_ADVANCED.md)
- [产品设计文档](https://github.com/multica-ai/multica/blob/main/docs/product-overview.md)
- [OpenClaw 官方仓库](https://github.com/openclaw/openclaw)
- [社区入门博客（OpenClaw API）](https://openclawapi.org/en/blog/2026-04-11-multica-ru-men)

---

## 中国用户适配

### 优先选自部署

`multica.ai` 云端依赖 Resend 邮件登录和 Google OAuth，国内访问不稳。**直接走自部署**——Apache 2.0 协议允许任意私有部署：

```bash
# 一键拉镜像 + 启动本地 server
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh | bash -s -- --with-server
multica setup self-host
```

如果 `ghcr.io` 拉镜像慢，从源码构建：

```bash
git clone https://github.com/multica-ai/multica.git
cd multica && make selfhost-build
```

自部署默认端口：前端 `http://localhost:3000`、后端 `http://localhost:8080`（可用 `multica setup self-host --port 9090 --frontend-port 4000` 改）。`.env` 必填项：

| 变量 | 说明 |
|------|------|
| `DATABASE_URL` | PostgreSQL 连接串 |
| `JWT_SECRET` | 用 `openssl rand -hex 32` 生成 |
| `FRONTEND_ORIGIN` | 前端 URL，用于 CORS |

**邮件登录是可选的**——`RESEND_API_KEY` 留空时，验证码会打印到 daemon stdout（或 `~/.multica/daemon.log`），照抄即可登录；本地开发场景把 `APP_ENV=development` 打开还能用万能验证码 `888888`（**仅限本机/内网，不要在公网实例上用**）。需要正式邮件登录时再去 [Resend](https://resend.com) 注册并填 `RESEND_API_KEY` 与 `RESEND_FROM_EMAIL`。

完整变量见 [SELF_HOSTING_ADVANCED.md](https://github.com/multica-ai/multica/blob/main/SELF_HOSTING_ADVANCED.md)。

### 飞书机器人通知

Multica 没有内置飞书通知，让 Agent 在完成任务后调用飞书自定义机器人。把 `$FEISHU_WEBHOOK_URL` 通过 Agent **Env Vars** 注入，然后在 **Instructions** 加一段：

```text
## 任务完成后推送飞书通知

当我分配的 Issue 进入终态（completed / failed）时，请：
1. 向 $FEISHU_WEBHOOK_URL 发一张 msg_type 为 "interactive" 的飞书卡片消息
2. 卡片正文需包含：Issue 标题、最终状态、分支或 PR 链接（如有）、
   80 字以内的工作摘要
3. 中间状态（queued / running / dispatched）不要发送通知
4. 失败时，把运行日志的最后一行错误信息附在卡片末尾
```

飞书自定义机器人创建步骤参考 [飞书 AI 助手](cn-feishu-ai-assistant.md#创建自定义机器人)。

### 国内 Provider 直连

Multica Provider 列表已包含 `kimi`（Moonshot Kimi CLI）和 `kiro-cli`（Kiro），两者国内可直连。在 Web UI 创建 Agent 时直接选这两个 Provider，可避开海外 API 出口。

---

**实测参考**：daemon 在一台 Mac mini 上稳定运行约 11 天（`uptime 261h`），自动检测到 `gemini / claude / codex / openclaw / hermes` 五个 Provider；同 Workspace 内并行 Claude / Codex / Hermes 三个 Agent，三条 Issue 在 `in_review` 状态正常流转，Web 看板与 `multica issue list` 输出实时一致。
