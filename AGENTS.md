# AGENTS.md

> **TL;DR (English)**: Chinese-language library of 50 verified personal-AI-agent use cases, written in **OpenClaw** terminology but usable by any agent. If you are an AI agent (Hermes / Claude Code / Codex / GPT), read this file first, then `INDEX.md` for the catalog and `CONCEPT-MAPPING.md` to translate OpenClaw terms to your agent's equivalents. Treat all use-cases as Markdown specs, not executable scripts. **Your job is not just to run a use case — it is to tell your human the fewest manual steps they must take, and when (see 人机分工协议 below).**

---

## 仓库定位

本仓库是面向中文用户的 **真实用例合集**，共 50 个经过社区验证的场景。用例目前以 OpenClaw 术语写就（Skill / Cron / Channel / SOUL.md / Memory / Sub-agent 等），但**不绑死 OpenClaw**——任何能读 Markdown 的 AI 智能体都能从中受益。仓库正在向「面向多 Agent 的领域最佳用例库」演进：让 Hermes / Claude Code / Codex 等都能快速学习并复用，并用一套统一的 to-agent 协议，引导各 agent 用最省事的方式协助自己的用户跑通。

## 你正在用哪个 Agent？（Quick start）

### OpenClaw（原生）
按 `usecases/<name>.md` 的"如何设置"步骤逐步执行，提示词可直接粘贴给你的 OpenClaw 实例。代码块执行规则见 [AGENT-GUIDE.md](AGENT-GUIDE.md)。

### Hermes（Nous Research）
如果 Hermes 从本仓库根目录启动，会读取本 `AGENTS.md`。如果你是在聊天界面或其他工作目录中使用 Hermes，落地路径：
- 先粘贴本 `AGENTS.md` 的 raw URL（协议不在用例文件里），再粘贴用例 raw URL 或文件路径，让 Hermes 拉取并解析；
- 将常用用例整理成 Hermes `SKILL.md` 后再安装；
- 已有 OpenClaw 配置可用 `hermes claw migrate` 导入。

### Claude Code（Anthropic）
`CLAUDE.md` 已指向本文件。把任意 `usecases/*.md` 当 task spec 提交，按下面的 Reading Protocol 执行；记忆和 sub-agent 概念见 `CONCEPT-MAPPING.md`。

### Codex / GPT Codex（OpenAI）
Codex 在开始工作前自动读取 `AGENTS.md`（CLI 是文档最完整的入口；Cloud / Web 同样遵循"最近的 AGENTS.md 优先"，并会沿目录树从根向下合并）。把用例文件路径或 URL 提供给 Codex 即可；遵守 Reading Protocol。

## Reading Protocol（执行协议）

执行任何用例前请遵守：

1. **Plan first**：先读完用例的"所需技能 → 如何设置 → 实用建议"全文，给出执行计划，列出会触达的外部系统。
2. **Dry-run**：能本地预演的步骤先空跑（生成草稿、打印命令而不执行），让用户确认输出无误。
3. **External writes 必须人工确认**：发邮件、发消息、发布内容、调用付费 API、修改远程仓库、转账或下单——执行前必须等用户明确授权。
4. **凭证占位符**：`YOUR_*` / `${VAR}` / `$VARIABLE` 由用户提供真实值；**永远不要**把凭证硬编码进配置文件，也**绝不亲手键入 / 写入凭证明文**——即使用户已把真实值给你，也只通过环境变量注入，命令和文件中只引用 `$VAR`。
5. **失败先报告**：报错时先停下汇报上下文，再尝试修复，不要默默重试或跳过安全检查（如 `--no-verify`）。
6. **Prompt 语言**：用例中的英文 prompt 通常效果最佳；中文版本（若有）见用例底部 "中国用户适配" 章节。
7. **不要修改本仓库案例**：除非用户明确要求贡献回上游，参考 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 人机分工协议（最重要）：先给用户"最小必要操作 + 时点"

本仓库的核心目标不只是"让 agent 跑通用例"，而是**让 agent 替用户想清楚：用最少的人工动作、在正确的时点，把用例跑起来**。执行任何用例前，你都要先产出一份"最小必要人工 + 时点"清单交给用户，再开始动手。

**第一步：查映射、推导人机分工。** 推导前先查 [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md)：把用例里每个 OpenClaw 能力词（Cron / Channel / Skill / Memory…）翻译成你自己的等价物，并记下你**没有原生等价物**的项——这些缺口必须原样写进第二步清单的「与原版差异」条目；**读过映射表却不把缺口告诉用户，视为违反本协议**。然后以用例正文为唯一事实来源，推导出**不变量**（授权 / 凭证 / 决策 / 付费门槛 / 外发确认，以及用户侧时点），这是必须交给用户判断的部分；而**能力项（装 skill / 写配置 / 建 cron 等）永远不推给用户**，无论用例怎么写，都由你现场探测后代办或回落。按下表从正文推导不变量：

| 槽位 | 在用例里怎么识别 | 输出给用户什么 |
|---|---|---|
| **一次性（开始前）** | "所需技能 / 安装 / 注册 / OAuth / `clawhub install` / 填 Key" | 按下方判定标准拆分：人工必做的部分给用户，其余你代办 |
| **周期性 / 自动** | "每天 / 每周 / 定时 / cron / heartbeat / schedule" | 之后由 cron 自动跑；仅在 token 续期等少数场景需人工 |
| **事件触发** | "当…时 / 收到…后 / 开会后 / 有新…" | 由什么事件触发，用户是否需在该时点配合 |
| **外发前确认** | "发送 / 发布 / 推送 / 付费 / 下单 / 转账 / 改远程" | 这些对外动作必须等用户确认（默认先 dry-run） |
| **凭证** | `$VAR` / `${VAR}` / `YOUR_*` / OAuth | 用户必须提供的真实值；**永不硬编码** |

**「必须人工」判定标准**：只有四类动作算人工必做——
① **身份 / 授权**：注册账号、OAuth、点邮件链接、扫码登录；② **凭证**：提供 / 批准 API Key、密码等真实值；③ **决策**：选平台、选方案、定监控对象等业务判断（可给带理由的推荐，但须停在用户确认点，不得把推荐当默认值径直推进；oneshot / 无法追问时也不例外）；④ **物理动作**：设备 / 线下操作。
其余——shell 安装（`npx` / `pip` / `brew`）、写文件、改配置、建 cron——**默认由你代办**。**回落证据分级**：你的工具列表里**有** shell / 执行工具时，只有**实际跑过**探测/安装命令（如 `which npx`、试跑安装）且它**真的报错**，才可把该动作移回"需用户协助"，并**贴出失败命令与报错原文**作为证据——严禁仅凭"本会话没有终端"这类自述偷懒回落；你的工具列表里**确实没有任何执行工具**（纯聊天界面）时，不必也**不得编造**试跑记录，如实声明『我的可用工具中没有 X，此项需你在可执行环境完成（或换用有 shell 的 agent）』即可回落。回退时按三段说清：『此项本应我代办 → 因〈证据：报错原文或工具缺失声明〉 → 故仅本轮请你在可写环境跑：…』，不得只甩裸命令清单（否则用户无法区分"环境硬限"与"你偷懒"）。
**自检（双向）**：① 把任何一项标为"需用户"之前，先答"它属于四类之一吗？"——不属于就探测后代办，不得以"界面要点击 / 看起来麻烦"为由推给用户；② 把任何③类业务决策自设默认值往下做之前，先答"用户给过真实值吗？"——没有就先问，不得用占位符代填后径直推进。

**第二步（动手前的硬门）：先只读探测，再把清单发给用户；清单发出并获回应前，不得写文件 / 改配置 / dry-run。**
① **只读探测**（仅探测、不写文件 / 不 dry-run）：实测自身能力——试写一个 /tmp 临时文件、`crontab -l`、列 MCP 工具、`which npx / node`——探到能定位卡点即可，不为"摸全环境"无界消耗（无执行工具的环境按上文回落证据分级处理）。探测放在清单之前，是为了让清单第 2、4 条写的是**你验证过的事实**，而不是先承诺、后验证。
② **发出清单**。它是事前契约、不是事后总结，也是你这一轮唯一的提问批次（之后只在"真卡住"时追加一次）：

> 这个用例要跑起来，你（人类）只需做这几件事：
> 1. **现在（一次性）**：……
> 2. **之后会自动**：……（无需你干预；只写你已探测确认能自动化的部分）
> 3. **只有这些需要你每次点头**：……（我会先给你草稿 / 预演；周期性外发在此声明授权级别，见下）
> 4. **与原版（OpenClaw）的差异**：我缺 ×××，将用 ××× 替代，你因此需要多做 ×××（据第一步的映射缺口如实填写；无差异则明写"无"）
>
> 需要你准备的凭证：……。其余交给我。首次跑通后我会实际验证一次并向你汇报，再交给自动化。

**外发授权只有三级**，周期性用例必须在清单第 3 条声明按哪一级执行并征询确认，不得自创语义（"发给用户本人"也是外发）：**每次确认**（默认——每次外发前给草稿等点头）／ **常设授权**（用户明确说过"这类外发以后不用再问"之后，同类外发自动执行，但首次仍须演示一次并留记录）／ **一次性**（仅本次有效，下次重新确认）。

**第三步：按"一次问齐、卡住才追加"的时序执行。**
① **一次性前置打包**：开跑前把所有 授权 / 凭证 / 决策 / 付费确认 一次问齐——不要做到一半才告知用户门槛（付费档、企业认证等前置阻塞依赖尤其如此）。其中**方向性③决策**（监控对象 / 数据源 / 索引目录 / 平台等"做什么"的选择）缺真实值时，**先问、不得用占位符或自选默认代填后继续**——占位符只能用于演示格式，不能替代征询；脚手架可先自办，但内容里的方向性选择留空待用户填，别替用户拍板；
② **自主执行其余**：装 skill / 写配置 / 建 cron 全程不打扰用户；
③ **真卡住才追加一次**；外发动作始终按清单声明的授权级别执行，未获常设授权的一律停在确认点。

> 这套推导规则的**设计目标**是对任何模型 / 任何 agent 通用：用例正文是唯一事实来源，不变量由你据此现场推导，能力项由你运行时探测，文件不预判。这一层"执行级适配"才是本仓库的价值所在。**已实测验证范围**：Codex（gpt-5.5，高 / 低推理档）与 Hermes 的真实运行对照；其余 agent 遵循同一协议但尚未系统实测，欢迎提交实测反馈。

## Where to Look

| 文件 | 用途 |
|---|---|
| [INDEX.md](INDEX.md) | 50 个用例的扁平索引：路径 / 一句话摘要 / 风险标签 |
| [CONCEPT-MAPPING.md](CONCEPT-MAPPING.md) | OpenClaw 术语 ↔ Hermes / Claude Code / Codex 等价物对照 |
| [AGENT-GUIDE.md](AGENT-GUIDE.md) | 用例文件结构与代码块执行规则的细则（人类与 agent 共用） |
| [CONTRIBUTING.md](CONTRIBUTING.md) | 贡献新用例的格式与收录标准 |
| `usecases/*.md` | 50 个用例本体，按文件名组织 |

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
- **不要** 把单个用例脱离本文件转发给其他 agent / 聊天界面——用例本体不含协议，先发（或同时发）本 `AGENTS.md`，否则这层适配对下游静默失效。
- **不要** 假设 SOUL.md 是当前会话的强制 system prompt——它是 OpenClaw 的人格定义文件，其他 agent 不一定有等价物（见 `CONCEPT-MAPPING.md`）。
- **不要** 因为某 OpenClaw 术语找不到对应就跳过用例，先查 `CONCEPT-MAPPING.md` 找等价物。
- **不要** 主动修改用例本体（改写步骤、加 frontmatter 等）——它们是稳定的人类阅读资产。
- **不要** 在脚本和配置中硬编码 API Key / token / 个人凭证。
- **不要** 用本合集中的用例去自动化抓取或群发到平台——社交媒体类用例都附带平台风控提醒，请遵守。
- **不要** 编造 OpenClaw / Hermes / Claude / Codex 不存在的功能或命令；不确定就在回复中明确指出并请用户验证。

---

> 本文件是仓库 AI agent 入口的单源真相。`CLAUDE.md` 仅作指针。如需扩展，请优先精简或下放到 `INDEX.md` / `CONCEPT-MAPPING.md`，保持本文件少于 150 行。
