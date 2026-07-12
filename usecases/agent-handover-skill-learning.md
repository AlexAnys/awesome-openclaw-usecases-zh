# 屏幕学习自动生成 AI 技能

> macOS 本地运行，无需云端

用 AgentHandover 观察你在 Mac 上的日常操作，自动提取工作流程并生成可复用的 AI Skill（智能体技能文件）。录制一次，永久移交——你的 Claude Code、OpenClaw、Codex 等智能体从此知道怎么替你干活，而且越用越准。

## 功能介绍

- **一键录制**：点击菜单栏 Record，命名任务，正常操作，点 Stop。系统问你 1-3 个澄清问题，自动生成完整 Skill
- **被动发现**：不用录制——正常工作即可。系统通过语义相似度（Semantic Similarity）识别重复模式，积累足够证据后自动生成 Skill
- **11 阶段智能管线**：截屏 → VLM 标注（视觉语言模型）→ 活动分类 → 文本向量化 → 图像向量化 → 语义聚类 → 跨会话关联 → 行为合成 → 声纹分析 → Skill 生成 → 人工审批
- **自我改进闭环**：智能体每次执行 Skill 后上报结果——成功提升信心分，偏差变成新的决策分支，连续失败自动降级
- **写作风格迁移**：系统学习你在不同场景的语气（Reddit 上随意、客户邮件正式），智能体按你的风格输出
- **六道安全门控**：生命周期审批 → 信任授权 → 新鲜度检查 → 预检条件 → 证据充分度 → 执行历史。没有你的批准，Skill 不会到达智能体

## 痛点

你每天在 Mac 上重复大量操作——社区运营、工单处理、数据提取、发布流程。你想把这些交给 AI 智能体，但手写 Skill 文件费时费力：要描述每一步、写决策逻辑、加护栏规则，而且流程一变就过时。更关键的是，手写 Skill 只能写"做什么"，写不出"为什么这么做"——选帖标准、跳过条件、措辞习惯这些隐性知识，全在你脑子里。

## 所需技能

- [AgentHandover](https://github.com/sandroandric/AgentHandover)（v0.2.1+）— 屏幕观察 + Skill 生成工具，macOS 原生应用
- [Ollama](https://ollama.com/) — 本地运行 VLM 推理引擎（AgentHandover 安装时自动引导下载模型）

可选：
- Chrome 浏览器 — 安装 AgentHandover Chrome 扩展后可捕获 DOM 快照、点击目标等浏览器上下文

## 如何设置

### 1. 安装 AgentHandover

从 [Releases 页面](https://github.com/sandroandric/AgentHandover/releases) 下载最新 `.pkg`，双击安装。引导程序会帮你完成：

- 系统权限授权（屏幕录制、辅助功能）
- 根据 Mac 内存自动推荐并下载最佳 AI 模型

| 内存 | 推荐模型 | 下载大小 |
|------|---------|---------|
| 8 GB | Qwen 3.5（2B + 4B） | ~6 GB |
| 16 GB | Gemma 4 E4B | ~10 GB |
| 24 GB | Gemma 4 E4B Q8 | ~12 GB |
| 48 GB+ | Gemma 4 31B | ~20 GB |

也可通过 Homebrew 安装：

```bash
brew tap sandroandric/agenthandover
brew install --HEAD agenthandover
```

安装完成后验证：

```bash
agenthandover doctor
```

### 2. 录制你的第一个工作流

点击菜单栏的 AgentHandover 图标，选择 **Record**，输入任务名称（如 "Daily Reddit marketing"），然后正常执行你的工作流程。完成后点击 **Stop**。

系统会从智能体的视角问你 1-3 个问题——例如 *"What determines which posts you engage with vs. skip?"*——然后自动生成 Skill。

也可以通过 CLI 录制：

```bash
# 开始录制
agenthandover focus start "Daily Reddit marketing"

# ...正常操作...

# 停止录制
agenthandover focus stop
```

### 3. 审批 Skill

在菜单栏应用中打开 **Workflows**，查看生成的 Skill 草稿。确认步骤、策略和护栏无误后，点击 **Approve for Agents**。

Skill 需要通过六道门控才能被智能体执行——这保证了你对每个自动化流程保有完全控制。

也可以通过 CLI 审批：

```bash
# 查看所有 Skill
agenthandover skills list

# 审批指定 Skill
agenthandover skills approve daily-reddit-marketing
```

### 4. 连接你的智能体

#### 方式一：MCP Server（推荐，适用于所有 MCP 兼容智能体）

在 OpenClaw 或 Claude Code 的 MCP 配置中添加：

```json
{
  "mcpServers": {
    "agenthandover": {
      "command": "agenthandover-mcp"
    }
  }
}
```

MCP Server 暴露 8 个工具，智能体可以列出、搜索、获取和执行 Skill，并上报执行结果触发自我改进。

#### 方式二：一键连接（自动写入配置）

```bash
# 连接 Claude Code — Skill 变成 /slash-command
agenthandover connect claude-code

# 连接 OpenClaw — Skill 自动同步到工作区
agenthandover connect openclaw

# 连接 Codex — 生成 AGENTS.md
agenthandover connect codex
```

### 5. 执行 Skill

连接完成后，在智能体中直接调用：

```text
/daily-reddit-marketing
```

智能体获取完整 Skill（步骤、策略、护栏、写作风格），按你的方式执行工作流。执行完成后自动上报结果，Skill 的信心分和决策分支随之更新。

## 生成的 Skill 长什么样

AgentHandover 生成的 Skill 比手写版本丰富得多——不只有步骤，还有策略、选择标准、护栏规则和写作风格：

```text
Reddit Community Marketing
Daily engagement workflow - 6 steps - 4 sessions learned

STRATEGY
Browse target subreddits for posts about marketing tools or growth
hacking. Engage with high-signal posts (10+ comments, posted within
48h, not promotional). Write authentic replies that acknowledge the
problem, share personal experience, and softly mention the product.

STEPS
1. Open Reddit and navigate to r/startups
2. Scan posts - skip promotional, skip < 10 comments
3. Open high-signal post and read top comments
4. Write reply: acknowledge -> experience -> mention product
5. Submit and verify not auto-removed
6. Repeat for r/marketing, r/growthacking (max 5/day)

SELECTION CRITERIA              GUARDRAILS
- Posts with 10+ comments       - Max 5 replies per day
- Not promotional or competitor - Never identical phrasing
- Posted within 48 hours        - Never reply to own posts
- Relevant to [product category]- Empathy-first tone always

VOICE & STYLE
Tone: casual | Sentences: short and punchy | Uses emoji

~15 min daily - 9-10am                     Confidence: 89%
```

手写 Skill 只能写出 STEPS。AgentHandover 额外提取了 STRATEGY（为什么这么做）、SELECTION CRITERIA（怎么选）、GUARDRAILS（什么不能做）和 VOICE & STYLE（怎么说话）——这些隐性知识来自对你真实操作的观察。

## 被动发现模式

如果你不想手动录制，可以开启 Passive Discovery（被动发现）模式——只需在菜单栏点击 **Observe Me**，然后正常工作。

系统在后台持续运行：
1. **截屏去重**：通过感知哈希（Perceptual Hashing）丢弃 ~70% 的重复帧
2. **VLM 标注**：本地视觉语言模型理解每一帧——什么应用、什么 URL、你在做什么
3. **语义聚类**：把相关操作按语义分组，"部署到 staging" 和 "push 到测试环境" 被识别为同一件事
4. **跨会话关联**：同一工作流在不同天的执行被关联起来
5. **行为合成**：积累 3 次以上观察后，自动提取策略、决策逻辑和模式，生成 Skill 草稿

你会在菜单栏收到通知：*"发现了一个新的重复工作流，点击查看草稿。"*

## 实用建议

- **从 Focus Recording 开始**：被动发现需要积累多次观察才能生成 Skill，Focus Recording 一次录制就能产出。建议先用 Focus Recording 把最常用的 3-5 个工作流录下来，再开启被动发现捕获长尾流程。
- **回答问题要具体**：录制结束后系统问你的 1-3 个问题直接决定 Skill 质量。回答"我选高互动帖子"不如回答"我选评论 10 条以上、48 小时内发布、非推广类的帖子"。
- **16 GB 内存体验最佳**：8 GB Mac 可以运行但标注速度较慢且模型能力有限。16 GB 以上搭配 Gemma 4 E4B 标注质量明显提升（v0.2.1 实测 Behavioral Confidence 从 0.20 跃升到 0.95）。
- **隐私完全本地**：所有推理在本地 Ollama 运行，截屏标注后即删，知识库加密存储（XChaCha20-Poly1305）。云端 API（OpenAI/Anthropic/Google）是 opt-in 的，默认不启用。
- **Skill 会过时——这是设计的一部分**：长时间未被观察到的 Skill 自动降级，防止智能体执行过时流程。重新执行一次或重新录制即可恢复。
- **用 `agenthandover watch` 实时监控**：在终端打开实时面板，查看截屏、标注、聚类的运行状态，排查问题比看日志更直观。
- **当前仅支持 macOS**：AgentHandover 依赖 macOS 的屏幕录制 API 和 SwiftUI 菜单栏应用，暂无 Windows/Linux 版本计划。

## 适用场景

AgentHandover 适合学习任何你在 Mac 上重复执行的工作流。以下是一些高价值场景：

| 场景 | 描述 |
|------|------|
| 社区运营 | Reddit/Discord/论坛的日常互动——选帖标准、回复风格、频率控制 |
| 工单处理 | 读工单 → 查后台 → 选模板 → 写回复的完整决策链 |
| 数据提取 | 从仪表盘、CRM、表格中按你的方式提取结构化数据 |
| 发布流程 | 部署、发布、状态更新——你的实际操作序列和决策点 |
| 调研流程 | 扫描信息源、提取关键事实、整理摘要的个人方法论 |

## 相关链接

- [AgentHandover GitHub](https://github.com/sandroandric/AgentHandover) — 源码、安装包、完整文档
- [AgentHandover Demo 视频](https://youtu.be/3nGH3rYbgfY) — 完整演示：录制 → 生成 Skill → 智能体执行
- [Ollama](https://ollama.com/) — AgentHandover 默认使用的本地 VLM 推理引擎
- [AI Agent Skills: The Complete Guide to SKILL.md](https://fungies.io/ai-agent-skills-skill-md-guide-2026/) — SKILL.md 格式详解，理解 AgentHandover 生成的文件格式
