# AI 首席参谋（Chief of Staff）

> 含国内适配：飞书/钉钉替代 Slack、Google Workspace 替代方案

用 ClawChief 开源操作系统把 OpenClaw 变成创始人的 AI 首席参谋。它每 15 分钟自动巡检邮件和日历，主动处理低风险事务、准备每日任务清单、跟进商务拓展线索，把原来需要一名全职行政助理（EA）的工作压缩到一套 cron 驱动的自动化流程中。

## 功能介绍

- **邮件分拣与自动回复**：每 15 分钟扫描收件箱，低风险邮件（日程确认、行政通知等）自动处理并归档，高风险邮件（法务、投资人、媒体）起草后等待审批
- **日历管理**：跨多个日历检查冲突，自动预约/改期/取消会议，解析 Booking Link（预约链接）直接下单
- **每日任务准备**：凌晨自动运行，把到期任务、日历会议、定期事项汇总为当天的优先级清单
- **商务拓展（BD）**：每天自动寻找潜在合作方，验证网站和邮箱有效性，发送初始外联邮件，更新 Google Sheets 追踪表
- **会议纪要摄取**：自动提取会议纪要中的行动项（Action Items），按优先级分类后写入任务系统
- **优先级路由**：所有信号经过 Priority Map（优先级映射）和 Auto-Resolver（自动决策层）两层过滤，决定"自动处理 / 起草等待审批 / 直接升级给老板"

## 痛点

创始人和高管每天被邮件、日历、跟进事项淹没。你可能试过雇一名行政助理（EA），但培训成本高、上下文丢失频繁、夜间和周末无人值守。或者你用了一堆 SaaS 工具拼凑自动化，但它们之间缺乏统一的决策逻辑——哪些邮件该立刻回、哪些该先问你、哪些可以直接归档？没有一个中央的"判断层"。

ClawChief 的核心设计理念是**分离关注点**：谁重要（Priority Map）、怎么处理（Auto-Resolver）、当前任务状态（tasks.md）、本地环境细节（TOOLS.md）各自独立，互不耦合。这样 AI 在每次执行时都能基于最新的源头数据做决策，而不是凭记忆猜测。

## 所需工具与技能

### 核心依赖

- [GOG CLI](https://github.com/steipete/gog) — Google Workspace 命令行工具（Gmail、Calendar、Sheets、Drive、Docs），ClawChief 的所有 Google 操作都通过它执行
- [Todoist](https://todoist.com/) — 任务管理后端（可选，ClawChief v2 已支持 Todoist 作为任务源）
- OpenClaw 内置的 cron 定时任务功能

### ClawChief 内置技能

| 技能 | 职责 |
|------|------|
| `executive-assistant` | 邮件分拣、日历管理、会议纪要摄取、日常行政 |
| `business-development` | 潜客开发、外联追踪、合作方跟进、CRM 更新 |
| `daily-task-prep` | 凌晨自动准备当天任务清单，归档昨日已完成项 |
| `daily-task-manager` | 响应用户的任务增删改查、优先级调整指令 |

## 如何设置

### 1. 安装 GOG CLI 并配置 Google OAuth

```bash
# 安装 GOG CLI
brew install steipete/tap/gogcli

# 加载 OAuth 客户端凭证（从 Google Cloud Console 下载的 JSON）
gog auth credentials /path/to/client_secret.json

# 添加助理邮箱账号并授权所需服务
gog auth add ${ASSISTANT_EMAIL} \
  --services gmail,calendar,drive,contacts,docs,sheets

# 验证
gog auth list
```

> 需要在 Google Cloud Console 中创建 OAuth Desktop 应用并启用 Gmail API、Google Calendar API、Google Sheets API、Google Drive API、Google Docs API、People API。详见仓库中的 `SETUP-GOG.md`。

### 2. 安装 ClawChief 技能和工作区文件

```bash
# 克隆仓库
git clone https://github.com/snarktank/clawchief.git
cd clawchief

# 复制技能到 OpenClaw 技能目录
cp -r skills/executive-assistant ~/.openclaw/skills/
cp -r skills/business-development ~/.openclaw/skills/
cp -r skills/daily-task-manager ~/.openclaw/skills/
cp -r skills/daily-task-prep ~/.openclaw/skills/

# 复制源数据层和工作区模板
cp -r clawchief/ ~/.openclaw/workspace/clawchief/
cp workspace/HEARTBEAT.md ~/.openclaw/workspace/
cp workspace/TOOLS.md ~/.openclaw/workspace/
mkdir -p ~/.openclaw/workspace/memory
cp workspace/memory/meeting-notes-state.json ~/.openclaw/workspace/memory/
```

### 3. 替换占位符并个性化

ClawChief 使用 `{{PLACEHOLDER}}` 模板变量，安装后需要全部替换为你的真实信息：

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `{{OWNER_NAME}}` | 你的名字 | `Alex` |
| `{{ASSISTANT_NAME}}` | AI 助理名称 | `R2` |
| `{{ASSISTANT_EMAIL}}` | 助理使用的 Gmail | `r2@yourcompany.com` |
| `{{PRIMARY_WORK_EMAIL}}` | 你的工作邮箱 | `alex@yourcompany.com` |
| `{{TIMEZONE}}` | 时区 | `Asia/Shanghai` |
| `{{BUSINESS_NAME}}` | 公司名 | `MyStartup` |
| `{{GOOGLE_SHEET_ID}}` | BD 追踪表 ID | `1BxiMVs0XRA...` |
| `{{TARGET_MARKET}}` | 目标客群 | `SaaS 创始人` |
| `{{TARGET_GEOGRAPHY}}` | 目标地域 | `中国大陆` |
| `{{PRIMARY_UPDATE_CHANNEL}}` | 推送渠道 | `slack` / `telegram` |
| `{{PRIMARY_UPDATE_TARGET}}` | 推送目标 | 频道 ID 或用户名 |

重点定制文件：

- `workspace/TOOLS.md` — 本地环境细节（邮箱、日历、BD 配置）
- `clawchief/priority-map.md` — 谁重要、什么项目重要、怎么路由
- `clawchief/tasks.md` — 当前任务状态
- `cron/jobs.template.json` — 定时任务模板

### 4. 定制 Priority Map

Priority Map 是 ClawChief 的"判断中枢"。编辑 `clawchief/priority-map.md`，定义：

```markdown
# 人员优先级示例

## 核心团队
### CTO 张三
- Watch for: 技术决策、架构阻塞、线上事故
- Escalate to P0 when: 生产环境故障或 24 小时内有硬性截止日期
- Default action: handle and summarize

## 项目优先级示例

### 首批 10 个付费客户
- Why it matters: 公司当前最高优先级
- Watch for: 合作方引荐、客户回复、转化瓶颈
- Escalate to P0 when: 热线索等待回复且时效敏感
```

### 5. 配置定时任务

使用 `cron/jobs.template.json` 作为模板创建 cron 任务：

```bash
# 核心三件套：
# 1. 行政助理巡检 — 每 15 分钟（工作时间 8:00-21:00）
# 2. 每日任务准备 — 凌晨 2:00
# 3. 商务拓展 — 凌晨 2:00（独立会话）
```

cron 任务示例（行政助理巡检）：

```json
{
  "name": "Executive assistant sweep",
  "enabled": true,
  "schedule": {
    "kind": "cron",
    "expr": "*/15 8-21 * * *",
    "tz": "Asia/Shanghai"
  },
  "sessionTarget": "main",
  "payload": {
    "kind": "systemEvent",
    "text": "Executive assistant sweep. Use the executive-assistant skill. Handle this reminder internally and only message the principal if there is new actionable information."
  }
}
```

### 6. 验证安装

按仓库中的 `INSTALL-CHECKLIST.md` 逐项验证：

```bash
# 验证 Gmail 搜索
gog gmail messages search -a ${ASSISTANT_EMAIL} \
  'in:inbox newer_than:7d' --max=5 --json --results-only

# 验证日历读取
gog calendar events --all -a ${ASSISTANT_EMAIL} \
  --days=2 --max=20 --json --results-only

# 验证 Sheets 读取
gog sheets metadata ${GOOGLE_SHEET_ID} \
  -a ${ASSISTANT_EMAIL} --json --results-only
```

## 真实案例

**场景一：15 分钟邮件巡检**

cron 触发行政助理巡检，OpenClaw 自动执行：

> **巡检完成 — 4 封处理，1 封需你决定**
>
> - [已处理] 与张总的会议改到周三下午 3 点，邀请已发送
> - [已处理] 供应商发票确认回复，已归档
> - [已处理] 团建投票链接，已转发给行政群
> - [已归档] 3 封营销邮件
> - [需决定] 投资人李总问能否下周三见面聊 Series A 进展，我起草了回复，请确认：
>   "李总您好，下周三下午 2-3 点方便，地点定在我们办公室如何？"

**场景二：凌晨任务准备**

每天凌晨 2 点自动运行 `daily-task-prep`：

> 把昨天已完成的 5 项任务归档到 `tasks-completed.md`，今天的任务清单已更新：
>
> **Ryan 的任务（按优先级排序）：**
> - P0: 回复李总关于 Series A 的邮件
> - P1: 10:00 与产品团队周会（已从日历同步）
> - P1: 14:00 合作方 Demo 演示
> - P2: 审核本周 BD 外联报告
>
> **R2 的任务：**
> - 跟进周一发出的 3 封未回复外联邮件
> - 为下周客户拜访预订会议室

**场景三：商务拓展自动化**

每天凌晨自动运行 BD 批次：

> **今日 BD 报告**
>
> - 新增 8 条已验证线索（网站正常 + 邮箱 MX 有效）
> - 发送 8 封初始外联邮件（使用 ryan@yourcompany.com）
> - 收到 2 封昨日外联的回复：
>   - 王律师事务所：有兴趣，已回复约下周二见面
>   - 李教练工作室：自动回复（已离职），标记为 not-a-fit
> - Google Sheets 追踪表已更新

## 架构设计精要

ClawChief 的核心设计模式是**关注点分离**（Separation of Concerns）：

```
clawchief/priority-map.md    → 谁重要 / 什么项目重要 / 紧急程度
clawchief/auto-resolver.md   → 自动处理 / 起草等审批 / 升级给老板
clawchief/meeting-notes.md   → 会议纪要摄取策略
clawchief/tasks.md           → 当前任务状态（唯一活跃源）
workspace/TOOLS.md           → 本地环境细节（邮箱、日历、BD 配置）
workspace/HEARTBEAT.md       → 编排层：每次心跳按什么顺序执行
cron/jobs.template.json      → 定时触发器
```

Auto-Resolver 的四种决策模式：

| 模式 | 触发条件 | 示例 |
|------|---------|------|
| 自动处理 | 低风险、可逆、权限明确 | 日程确认、任务更新、行政邮件归档 |
| 起草等审批 | 判断归老板、措辞敏感 | 投资人回复、法务问题、公开发言 |
| 升级不动 | 权限不明、风险高 | 合同条款、矛盾信息、隐私数据 |
| 忽略/归档 | 噪音、重复、无需行动 | 营销邮件、重复通知 |

## 实用建议

- **从行政助理技能开始**：先只跑 `executive-assistant` 的 15 分钟巡检，确认邮件分拣和日历管理稳定后再开启 BD 技能。循序渐进比一步到位更安全。
- **Priority Map 是核心**：花时间把你的关键人物和项目写进 Priority Map。这不是可选配置——它直接决定 AI 的判断质量。Ryan Carson 的原始 Priority Map 超过 400 行，覆盖了家人、投资人、合作方、各个业务线。
- **用 message-level 搜索而非 thread-only**：ClawChief 强调 Gmail 搜索要用消息级别（message-level）而非仅线程级别，否则会漏掉关键回复。
- **同一轮更新源数据**：每次自动处理后必须在同一执行轮次中更新对应的源数据（任务文件、追踪表等），不要留到下次。这是保持数据一致性的铁律。
- **回复保持线程上下文**：用 `--reply-to-message-id` 回复已有邮件，而不是新建一封带 `Re:` 前缀的邮件，否则会破坏邮件线程。
- **从窄到宽**：BD 技能初期只针对一个细分市场、一个地域，验证外联质量后再扩展范围。

## 中国用户适配

### Google Workspace 替代方案

ClawChief 深度依赖 Google Workspace（Gmail + Calendar + Sheets）。国内用户如果无法使用 Google 服务，需要替换底层工具层：

| 原始工具 | 国内替代 | 说明 |
|---------|---------|------|
| Gmail（GOG CLI） | 飞书邮箱 / 企业微信邮箱 / 阿里邮箱 | 需要对应的 API 或 MCP 工具 |
| Google Calendar | 飞书日历 / 钉钉日历 | 通过飞书/钉钉开放 API 读写 |
| Google Sheets | 飞书多维表格 / 钉钉宜搭 | 作为 BD 追踪表的替代 |
| Slack（推送渠道） | 飞书机器人 / 钉钉机器人 / 企业微信机器人 | Webhook 推送巡检结果 |

### 飞书生态适配路径

对于使用飞书的团队，推荐参考本仓库的飞书相关用例：

- 使用 [Lark CLI MCP](https://github.com/nicobailon/lark-cli-mcp) 作为飞书的命令行操作工具，类似 GOG 之于 Google Workspace
- 邮件分拣：通过飞书邮箱 API 实现收件箱扫描和自动回复
- 日历管理：通过飞书日历 API 查冲突、创建/修改日程
- 任务追踪：飞书多维表格替代 Google Sheets，或直接使用飞书任务

### 推送渠道适配

将巡检结果和 BD 报告推送到国内团队工具：

```text
After completing the executive-assistant sweep or business-development batch,
send a summary to the team:
- Format key findings as a structured message
- Include items needing principal decision at the top
- Use Feishu webhook: ${FEISHU_WEBHOOK_URL}
```

飞书 Webhook 推送示例：

```bash
curl -X POST "${FEISHU_WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "interactive",
    "card": {
      "header": {
        "title": {"tag": "plain_text", "content": "EA 巡检报告 - 2026-04-11"},
        "template": "blue"
      },
      "elements": [
        {
          "tag": "markdown",
          "content": "**需决定** 投资人李总约下周三\n**已处理** 3 封日程确认 + 2 封归档"
        }
      ]
    }
  }'
```

### 适配注意事项

- ClawChief 的核心价值不在具体工具（GOG / Gmail），而在**架构设计**：Priority Map + Auto-Resolver + 关注点分离。这套模式可以完整移植到飞书/钉钉生态
- 即使不替换 Google Workspace，仍可以学习 ClawChief 的 Priority Map 和 Auto-Resolver 设计模式，应用到任何 OpenClaw 工作流中
- 国内创业者可以把 BD 技能中的"律师/教练"目标客群替换为自己的业务场景

## 与其他用例的区别

| 维度 | AI 首席参谋（本用例） | 收件箱整理 | 会议纪要提取 | 个人 CRM |
|------|---------------------|-----------|-------------|---------|
| 范围 | 邮件 + 日历 + 任务 + BD 全栈 | 仅邮件分类 | 仅会议 → 行动项 | 仅联系人关系 |
| 决策层 | Priority Map + Auto-Resolver 双层策略 | 规则或标签 | 无 | 无 |
| 执行模式 | cron 驱动，每 15 分钟主动巡检 | 被动触发 | 会后触发 | 手动触发 |
| 自动处理 | 低风险事务自动执行并更新源数据 | 仅分类 | 仅提取 | 仅记录 |
| 适用对象 | 创始人/高管/需要"参谋"级支持的人 | 所有人 | 会议多的人 | 社交活跃的人 |

## 相关链接

- [ClawChief 仓库](https://github.com/snarktank/clawchief) — 完整的技能文件、策略模板和安装文档
- [Ryan Carson 的发布帖](https://x.com/ryancarson/status/2039786704731541903) — 原始发布和实战演示
- [This Week in Startups E2272](https://podcasts.apple.com/us/podcast/3-ai-agents-that-actually-replaced-human-jobs-e2272/id315114957?i=1000759945461) — ClawChief 在 TWIS 播客上的 Demo 展示
- [GOG CLI](https://github.com/steipete/gog) — Google Workspace 命令行工具
- [Pedro Franceschi 的 Core Memory 播客片段](https://www.youtube.com/watch?v=9ZbbxSgrjhw&t=3847s) — ClawChief 设计灵感来源
- [bdougie 的架构评审 Gist](https://gist.github.com/bdougie/b7c979437372011a4785c5a01720b2a2) — GitHub 员工对 ClawChief 架构的独立评审

---

**原始来源**：[snarktank/clawchief](https://github.com/snarktank/clawchief)（738 stars，89 forks，2026 年 4 月 2 日发布，Ryan Carson 实际运行于其创业公司日常运营中）
