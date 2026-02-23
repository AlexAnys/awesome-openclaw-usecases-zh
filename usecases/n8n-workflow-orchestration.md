# OpenClaw + n8n 工作流编排

让你的 AI 智能体（agent）直接管理 API 密钥和调用外部服务，是安全事故的温床。每增加一个新集成，就意味着 `.env.local` 中多一个凭据，多一个智能体可能意外泄露或误用的攻击面。

这个用例描述了一种模式：OpenClaw 通过 webhook（网络钩子）将所有外部 API 交互委托给 n8n 工作流——智能体永远不接触凭据，每个集成都可以可视化检查和锁定。

## 痛点

当 OpenClaw 直接处理所有事情时，你会面临三个叠加的问题：

- **缺乏可见性**：当逻辑埋在 JavaScript 技能文件或 shell 脚本中时，很难检查智能体实际构建了什么
- **凭据蔓延**：每个 API 密钥都存在于智能体的环境中，一次错误的提交就可能暴露
- **浪费令牌（token）**：确定性子任务（发送邮件、更新电子表格）本可以作为简单工作流运行，却在消耗 LLM 推理令牌

## 功能概述

- **代理模式**：OpenClaw 编写带有传入 webhook 的 n8n 工作流，然后通过调用这些 webhook 进行所有后续的 API 交互
- **凭据隔离**：API 密钥存储在 n8n 的凭据仓库中——智能体只知道 webhook URL
- **可视化调试**：每个工作流都可以在 n8n 的拖拽式 UI 中检查
- **可锁定工作流**：工作流构建和测试完成后，你可以锁定它，防止智能体修改其与 API 的交互方式
- **安全防护步骤**：你可以在 n8n 中添加验证、速率限制和审批门控，在任何外部调用执行前进行拦截

## 工作原理

1. **智能体设计工作流**：告诉 OpenClaw 你需要什么（例如，"创建一个工作流，当新的 GitHub issue 被标记为 `urgent` 时发送 Slack 消息"）
2. **智能体在 n8n 中构建**：OpenClaw 通过 n8n 的 API 创建工作流，包括传入 webhook 触发器
3. **你添加凭据**：打开 n8n 的 UI，手动添加你的 Slack token / GitHub token
4. **你锁定工作流**：防止智能体进一步修改
5. **智能体调用 webhook**：从此以后，OpenClaw 使用 JSON 载荷调用 `http://n8n:5678/webhook/my-workflow`——它永远看不到 API 密钥

```text
┌──────────────┐     webhook 调用      ┌─────────────────┐     API 调用    ┌──────────────┐
│   OpenClaw   │ ───────────────────→  │   n8n 工作流     │ ─────────────→  │   外部服务    │
│  （智能体）   │   （无凭据）           │ （已锁定，含     │  （凭据留在     │ （Slack 等）  │
│              │                       │   API 密钥）     │   此处）        │              │
└──────────────┘                       └─────────────────┘                  └──────────────┘
```

## 所需技能

- `n8n` API 访问（用于创建/触发工作流）
- `fetch` 或 `curl`，用于 webhook 调用
- Docker（如果使用预配置的技术栈）
- n8n 凭据管理（手动操作，每个集成只需设置一次）

## 如何设置

### 方案一：预配置的 Docker 技术栈

一个社区维护的 Docker Compose 配置（[openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack)）在共享 Docker 网络上预先连接了所有组件：

```bash
git clone https://github.com/caprihan/openclaw-n8n-stack.git
cd openclaw-n8n-stack
cp .env.template .env
# 将你的 Anthropic API 密钥添加到 .env
docker-compose up -d
```

这将为你提供：
- OpenClaw 运行在端口 3456
- n8n 运行在端口 5678
- 共享 Docker 网络，使 OpenClaw 可以直接调用 `http://n8n:5678/webhook/...`
- 预构建的工作流模板（多 LLM 事实核查、邮件分类、社交监控）

### 方案二：手动设置

1. 安装 n8n（`npm install n8n -g` 或通过 Docker 运行）
2. 配置 OpenClaw 使其知道 n8n 的基础 URL
3. 将以下内容添加到你的 AGENTS.md：

```text
## n8n Integration Pattern

When I need to interact with external APIs:

1. NEVER store API keys in my environment or skill files
2. Check if an n8n workflow already exists for this integration
3. If not, create one via n8n API with a webhook trigger
4. Notify the user to add credentials and lock the workflow
5. For all future calls, use the webhook URL with a JSON payload

Workflow naming: openclaw-{service}-{action}
Example: openclaw-slack-send-message

Webhook call format:
curl -X POST http://n8n:5678/webhook/{workflow-name} \
  -H "Content-Type: application/json" \
  -d '{"channel": "#general", "message": "Hello from OpenClaw"}'
```

## 关键洞察

- **一举三得**：可观测性（可视化 UI）、安全性（凭据隔离）和性能（确定性工作流不消耗令牌）
- **测试后锁定**：「构建 → 测试 → 锁定」的循环至关重要——不锁定的话，智能体可以悄悄修改工作流
- **n8n 拥有 400+ 集成**：你想连接的大多数外部服务已经有 n8n 节点，省去了智能体编写自定义 API 调用的工作
- **免费的审计追踪**：n8n 记录每次工作流执行的输入/输出数据

## 灵感来源

这个模式由 [Simon Hoiberg](https://x.com/SimonHoiberg/status/2020843874382487959) 描述，他阐述了这种方法优于让 OpenClaw 直接处理 API 交互的三个原因：通过 n8n 可视化 UI 实现可观测性、通过凭据隔离实现安全性、以及通过将确定性子任务作为工作流而非 LLM 调用来运行以提升性能。[openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack) 仓库提供了一个即用型的 Docker Compose 配置来实现此模式。

## 相关链接

- [n8n 文档](https://docs.n8n.io/)
- [openclaw-n8n-stack（Docker 配置）](https://github.com/caprihan/openclaw-n8n-stack)
- [n8n Webhook 触发器文档](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/n8n-workflow-orchestration.md)
