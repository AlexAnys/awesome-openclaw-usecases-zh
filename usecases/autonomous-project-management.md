# 使用子智能体（Sub-agent）进行自主项目管理

管理包含多个并行工作流的复杂项目令人精疲力竭。你不得不频繁切换上下文、在不同工具间追踪状态，并手动协调任务交接。

本用例实现了一种去中心化的项目管理模式，其中子智能体（sub-agent）自主处理任务，通过共享状态文件而非中央调度器进行协调。

## 痛点

传统的调度器模式会造成瓶颈——主智能体变成了"交通警察"。对于复杂项目（多仓库重构、研究冲刺、内容管道），你需要能够并行工作且无需持续监督的智能体。

## 功能概述

- **去中心化协调**：智能体通过共享的 `STATE.yaml` 文件进行读写协调
- **并行执行**：多个子智能体同时处理独立任务
- **无调度器开销**：主会话保持精简（CEO 模式——仅负责策略）
- **自文档化**：所有任务状态持久化在版本控制文件中

## 核心模式：STATE.yaml

每个项目维护一个 `STATE.yaml` 文件作为唯一的事实来源：

```yaml
# STATE.yaml - 项目协调文件
project: website-redesign
updated: 2026-02-10T14:30:00Z

tasks:
  - id: homepage-hero
    status: in_progress
    owner: pm-frontend
    started: 2026-02-10T12:00:00Z
    notes: "Working on responsive layout"

  - id: api-auth
    status: done
    owner: pm-backend
    completed: 2026-02-10T14:00:00Z
    output: "src/api/auth.ts"

  - id: content-migration
    status: blocked
    owner: pm-content
    blocked_by: api-auth
    notes: "Waiting for new endpoint schema"

next_actions:
  - "pm-content: Resume migration now that api-auth is done"
  - "pm-frontend: Review hero with design team"
```

## 工作原理

1. **主智能体接收任务** → 生成具有特定范围的子智能体
2. **子智能体读取 STATE.yaml** → 找到分配给自己的任务
3. **子智能体自主工作** → 更新 STATE.yaml 中的进度
4. **其他智能体轮询 STATE.yaml** → 认领已解除阻塞的工作
5. **主智能体定期检查** → 审查状态，调整优先级

## 所需技能

- `sessions_spawn` / `sessions_send` 用于子智能体管理
- 文件系统访问以操作 STATE.yaml
- Git 用于状态版本控制（可选但推荐）

## 设置：AGENTS.md 配置

```text
## PM 委派模式

主会话 = 仅作为协调者。所有执行交给子智能体。

工作流程：
1. 新任务到达
2. 检查 PROJECT_REGISTRY.md 查找现有 PM
3. 如果 PM 存在 → sessions_send(label="pm-xxx", message="[task]")
4. 如果是新项目 → sessions_spawn(label="pm-xxx", task="[task]")
5. PM 执行任务，更新 STATE.yaml，汇报结果
6. 主智能体向用户总结

规则：
- 主会话：最多 0-2 次工具调用（仅 spawn/send）
- PM 拥有自己的 STATE.yaml 文件
- PM 可以生成子子智能体处理并行子任务
- 所有状态变更提交到 git
```

## 示例：生成一个 PM

以下是一个用户请求和智能体响应的示例：

```text
User: "Refactor the auth module and update the docs"

Main agent:
1. Checks registry → no active pm-auth
2. Spawns: sessions_spawn(
     label="pm-auth-refactor",
     task="Refactor auth module, update docs. Track in STATE.yaml"
   )
3. Responds: "Spawned pm-auth-refactor. I'll report back when done."

PM subagent:
1. Creates STATE.yaml with task breakdown
2. Works through tasks, updating status
3. Commits changes
4. Reports completion to main
```

## 关键洞察

- **STATE.yaml 优于调度器**：基于文件的协调比消息传递更具扩展性
- **Git 作为审计日志**：提交 STATE.yaml 的变更以获得完整历史记录
- **标签命名规范很重要**：使用 `pm-{project}-{scope}` 格式便于追踪
- **精简主会话**：主智能体做得越少，响应就越快

## 灵感来源

该模式受到 [Nicholas Carlini 的方法](https://nicholas.carlini.com/)启发——让智能体自我组织，而非微观管理它们。

## 相关链接

- [OpenClaw 子智能体文档](https://github.com/openclaw/openclaw)
- [Anthropic：构建高效智能体](https://www.anthropic.com/research/building-effective-agents)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/autonomous-project-management.md)
