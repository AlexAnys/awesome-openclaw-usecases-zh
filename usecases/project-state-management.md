# 项目状态管理系统：替代看板的事件驱动方案

传统看板（Kanban）是静态的，需要手动更新。你会忘记移动卡片，在不同会话之间丢失上下文，而且无法追踪状态变更背后的"原因"。项目在缺乏清晰可见性的情况下逐渐偏离轨道。

这个工作流用事件驱动（event-driven）系统替代看板，自动跟踪项目状态：

- 将项目状态存储在数据库中，保留完整历史记录
- 捕获上下文：决策、阻塞项、下一步计划、关键洞察
- 事件驱动更新："刚完成 X，被 Y 阻塞" -> 自动状态转换
- 自然语言查询："[项目]的状态是什么？"、"我们为什么在[功能]上转向了？"
- 每日站会摘要：昨天做了什么，今天计划什么，什么被阻塞了
- Git 集成：将提交记录链接到项目事件，实现可追溯性

## 痛点

看板容易过时。你把时间浪费在更新卡片上，而不是做实际工作。上下文会丢失——三个月后，你已经想不起当初为什么做了一个关键决策。代码变更和项目进展之间没有自动关联。

## 功能介绍

你不再需要拖动卡片，而是直接和助手对话："完成了认证流程，开始做仪表盘。" 系统会记录事件、更新项目状态并保留上下文。当你问"仪表盘进展如何？"时，它会给你完整的故事：什么已完成、下一步是什么、什么在阻塞你，以及原因。

Git 提交记录会被自动扫描并关联到项目。你的每日站会摘要会自动生成。

## 所需技能

- `postgres` 或 SQLite 用于项目状态数据库
- `github`（gh CLI）用于提交记录跟踪
- Discord 或 Telegram 用于更新和查询
- 定时任务（cron job）用于每日摘要
- 子智能体（sub-agent）用于并行项目分析

## 设置方法

1. 设置项目状态数据库：
```sql
-- 项目表
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name TEXT UNIQUE,
  status TEXT, -- 例如 "active"（进行中）、"blocked"（阻塞）、"completed"（已完成）
  current_phase TEXT,
  last_update TIMESTAMPTZ DEFAULT NOW()
);

-- 事件表
CREATE TABLE events (
  id SERIAL PRIMARY KEY,
  project_id INTEGER REFERENCES projects(id),
  event_type TEXT, -- 例如 "progress"（进展）、"blocker"（阻塞）、"decision"（决策）、"pivot"（转向）
  description TEXT,
  context TEXT,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

-- 阻塞项表
CREATE TABLE blockers (
  id SERIAL PRIMARY KEY,
  project_id INTEGER REFERENCES projects(id),
  blocker_text TEXT,
  status TEXT DEFAULT 'open', -- "open"（未解决）、"resolved"（已解决）
  created_at TIMESTAMPTZ DEFAULT NOW(),
  resolved_at TIMESTAMPTZ
);
```

2. 创建一个用于项目更新的 Discord 频道（例如 #project-state）。

3. 给 OpenClaw 设置提示词：

以下提示词告诉智能体如何根据你的对话自动管理项目状态：

```text
You are my project state manager. Instead of Kanban, I'll tell you what I'm working on conversationally.

When I say things like:
- "Finished [task]" → log a "progress" event, update project state
- "Blocked on [issue]" → create a blocker entry, update project status to "blocked"
- "Starting [new task]" → log a "progress" event, update current phase
- "Decided to [decision]" → log a "decision" event with full context
- "Pivoting to [new approach]" → log a "pivot" event with reasoning

When I ask:
- "What's the status of [project]?" → fetch latest events, blockers, and current phase
- "Why did we decide [X]?" → search events for decision context
- "What's blocking us?" → list all open blockers across projects

Every morning at 9 AM, run a cron job to:
1. Scan git commits from the past 24 hours (via gh CLI)
2. Link commits to projects based on branch names or commit messages
3. Post a daily standup summary to Discord #project-state:
   - What happened yesterday (events + commits)
   - What's planned today (based on current phase and recent conversations)
   - What's blocked (open blockers)

When I'm planning a sprint, spawn a sub-agent to analyze each project's state and suggest priorities.
```

4. 集成到你的工作流中：只需自然地和助手聊你正在做的事情，系统会自动捕获一切。

## 相关链接

- [事件溯源模式（Event Sourcing Pattern）](https://martinfowler.com/eaaDev/EventSourcing.html)
- [为什么看板不适合独立开发者](https://blog.nuclino.com/why-kanban-doesnt-work-for-me)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/project-state-management.md)
