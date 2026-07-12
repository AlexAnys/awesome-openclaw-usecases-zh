<!-- 本文件由多 Agent 研究工作流生成（2026-06），经对抗式核验与去重。候选用例均未由本仓库维护者亲自跑通，**不构成收录**——按 CONTRIBUTING 的“真实跑通 + 多源验证 + 可复现”红线，需先验证一次真实运行后才可成稿入库。 -->

# Reverse-adaptation analysis：旗舰用例 → 通用模式 → 跨 Agent 适配性

> 数据范围：flagship-claude-code / flagship-codex / flagship-other-frameworks 三条 mining lane 共 13 个候选。本分析基于 screen 结论标注每个候选的可信度（`passes_bar` + `confidence` + `real_user_confirmed`），并据此判断「值不值得 port 进本库」。
> 关键原则：本库收录标准要求 **真实跑通 + 多源验证 + 读者可复现**，且 reverse-adaptation（把 Claude Code / Codex / Hermes 原生用例改写成 agent-agnostic / OpenClaw 用例）本身 **几乎都是 contributor 的合成假设，没有任何来源真的在目标 agent 上跑过**。下面把「已验证的原生事实」和「未验证的反向移植」严格分开。

## 1. 旗舰用例 → 通用模式 → 各 Agent 跑通所需能力

下表中「✅」= 该 agent 有干净对应原语；「⚠️」= 能拼出来但需自建胶水/无一等公民；「❌」= 无干净对应。`Claude Code` 列在原生用例上标 native。

| # | 旗舰原生用例（lane / 验证状态） | 可泛化的核心模式 | OpenClaw 需要什么 | Codex 需要什么 | Hermes 需要什么 | 谁没有干净对应 |
|---|---|---|---|---|---|---|
| **A** | **单会话 Subagent 扇出做大规模语言/框架迁移**（Dynamic Workflows，Bun Zig→Rust，PR #30412 真实合并）<br>claude-code · passes ✅ · conf 0.82 · real ✅ | 编排器在**一个会话内**写运行时计划 → 扇出几十~几百个**短命文件级 subagent**（映射生命周期→逐文件移植→每文件双 reviewer→build/test 修复环→清理 PR），带 **resumable checkpoint** 断点续跑 | `sessions_spawn` 批量起短命子会话 + 一份 porting-rules 文档 + STATE 文件做 checkpoint + 双 reviewer 子会话做 merge gate。⚠️ OpenClaw 原生是「fleet/团队」模型（长命 agent 各自一个 PR），**单会话内瞬时扇出几百 subagent + JS 计划编排不是一等公民**，需用 cron/state 自建 | ⚠️ 有 headless CLI + AGENTS.md 规则，可逐文件跑，但**无内建的单会话内并行 subagent 扇出原语**，并行靠外部脚本起多个 codex 进程 | ⚠️ 有 subagent / skills，可写 porting skill，但**百级并行 + checkpoint 续跑无现成 harness** | **Claude Code 独有**：单会话内「写 JS 计划 → 几百并行 ephemeral subagent → 双 reviewer 逐文件 gate」是其 Dynamic Workflows 的一等公民；其他 agent 全要自建编排层 |
| **B** | **隔夜自愈自治循环**（STATUS.md handoff + watchdog + headless `--print`，evekhm 公开 repo）<br>claude-code · passes ✅ · conf 0.72 · real ✅ | bash 顺序起**全新无上下文 headless 会话**，每个读/写一份 `STATUS.md` 交接文件替代会话上下文；分阶段 `/goal` 完成条件；watchdog 失败从下一 checkpoint 重启；CLAUDE.md 强制命令输出落盘只读 tail 摘要以避免上下文溢出 | ✅ **几乎一一对应**：`cron` + `sessions_spawn`（每次 fresh context）+ `file_read`/`file_write` 维护 STATUS/STATE handoff + watchdog 用 cron 巡检重启。这是适配性最干净的一个 | ✅ headless `codex exec` + AGENTS.md 输出落盘规则 + 外部 watchdog 脚本，可直接复刻 | ✅ 长命 agent + cron + skill 文件做 handoff，机制等价 | 无明显短板。**模式本身 agent-agnostic**，三家都能跑；唯一门槛是「输出落盘 + fresh-context 顺序会话」这套纪律 |
| **C** | **Headless CI Agent：事件驱动 Issue→PR / 修测试 / 自动 changelog**（claude-code-action，官方 repo 真实；但唯一第一人称叙述者是 AI 营销 persona）<br>claude-code · **fail** ❌ · conf 0.82 · real ❌ | agent 完全活在 CI 内（零本地足迹、触发时零人在环），由仓库 GitHub 事件触发：评论触发草稿 PR、CI 失败分析 artifact 定位根因、tag 触发 changelog、label 触发 spec→stub | webhook/`RemoteTrigger`（POST /hooks/wake）唤醒 headless 会话 + repo 写权限 + gh CLI + 成本护栏。⚠️ OpenClaw 有 wake webhook，但「GitHub 事件→具体 4 工作流配方」需自接 | ✅ Codex 可在 CI runner 内 headless 跑 gh 循环（官方有 GitLab/CI 示例），但具体 4 配方需自写 | ⚠️ 需自建 webhook 接入 + CI 编排，无现成 GitHub Action | **概念无独家**，三家原语都在；但 ai-scaffold label 流 / CI-artifact 根因流**没有任何真人跑通证据** |
| **D** | **Agent 循环内的自动安全审查 gate**（security-guidance 插件 + claude-code-security-review，官方真实）<br>claude-code · **fail** ❌ · conf 0.78 · real ❌ | 每次文件编辑跑**无模型调用的确定性 pattern-match**（eval/exec/pickle/DOM 注入）+ turn 级模型审查 + PR 级 Action diff 扫描，作为对 agent 自身产出的安全检查 | pre-write hook + pre-merge 子 agent 审查步骤。⚠️ OpenClaw 有 hook 机制可接，但**「pre-write hook」适配是 screener 发明，无人跑过** | ⚠️ Codex 可在 commit/CI 接扫描，但 in-session 逐编辑 hook 非一等公民 | ⚠️ 同上，需自建 hook | **重要纠错**：官方 in-session 插件**不 block**（仅 advisory），只有独立 CI Action 能 fail-on-findings；候选把两者混成「PR-blocking gate」是 false-confidence 错误。Hermes/OpenClaw 的 pre-write **硬 gate** 无干净对应 |
| **E** | **自托管 Codex GitLab MR 审查 bot（安全加固）**（Sleeyax dev.to，闭源）<br>codex · **fail** ❌ · conf 0.83 · real ✅(但闭源不可复现) | GitLab webhook → Fastify → BullMQ/Redis → worker → codex-sdk 流水线，发内联评论、JSONL 续跑会话；**danger-full-access 沙箱加固playbook**（多容器隔离凭证、seccomp 拦 AF_UNIX、4 层 egress 锁定、临时 authed git URL 仅作 argv） | ⚠️ codex-sdk 服务与 OpenClaw 无关；**沙箱/egress/凭证隔离 playbook 可映射到 OpenClaw skills + sandboxing**，但这是未测试的 port | ✅ 原生就是 Codex（codex-sdk headless），但**整套闭源、bespoke OpenShift 配置不可复现** | ⚠️ 可换 Hermes 跑审查，沙箱 playbook 通用 | **闭源 + 单源**导致整体不可复现；但「在 danger-full-access 下安全跑 agent 的沙箱 playbook」是全库独一份、值得提取的横切知识 |
| **F** | **对抗式多 agent bug hunter + 安全分支自动修**（codexstar69/bug-hunter，402★，多 CLI）<br>codex · **fail** ❌ · conf 0.8 · real ❌ | Triage→Recon→Hunter→Skeptic→Referee→Fix→Verify 对抗流水线：一个 agent 找 bug，Skeptic 反驳，Referee 给 CVSS 裁决，Fixer 在 git 分支逐修提交 + 测试基线 + 失败自动回滚 + 修后复扫。**靠「发现需活过三阶段」压低误报** | Hunter/Skeptic/Referee 拆成 OpenClaw 子 agent + git 分支 canary。⚠️ 角色拆分干净可移植，但无真人跑通 | ✅ 仓库明确支持 Codex CLI | ⚠️ 可拆成 Hermes subagent | **模式 agent-agnostic（仓库自称支持 7 种 CLI）**；短板是**零第三方端到端运行证据**（单作者、0 issue） |
| **G** | **OSS 维护即可复用 skills（AGENTS.md + GitHub Actions）**（OpenAI 自己维护 Agents SDK，repo 可查 + +45% PR 数据）<br>codex · **passes** ✅ · conf 0.82 · real ✅ | 把反复出现的维护杂活固化成 repo-local skills（code-change-verification / docs-sync / examples-auto-run / release-review / pr-draft-summary…），在 **AGENTS.md 声明强制触发**，由 GitHub Actions 调用 | ✅ OpenClaw skills 目录 + 触发声明 + CI 调用，机制干净对应（skill 概念几乎平移） | ✅ **原生**（OpenAI 自用就是 Codex + AGENTS.md + run.sh/run.ps1） | ✅ Hermes skills 系统直接对应 | 无明显短板。**skill 概念三家通用**；唯一 caveat：必须标注「OpenClaw 映射是 contributor 改写，OpenAI 并未在 OpenClaw 上用」 |
| **H** | **发布前全仓 Codex 安全/质量审计 ship gate（advisory P0-P3 + go/no-go）**（UncleTIM repo 真实 dated artifact）<br>codex · **fail** ❌ · conf 0.72 · real ✅(但单源 + 私有 runtime) | Codex 只读对抗审查整个包面（src/tests/docs/examples/CI/package.json/npm+MCP 边界）→ 结构化 P0-P3 报告（不打补丁）+ go/no-go 裁决，并发竞态/锁绕过写路径在外部用户命中前暴露 | ⚠️ 可用 OpenClaw 只读会话 + Ralph-style audit 环写 `audit/*.md` 复刻，但需重写 | ✅ Codex CLI `/review` 原生支持只读审查 | ⚠️ 可换只读审查 agent | 不可复现根因：方法依赖作者**私有 codex-companion runtime**，读者无公开命令可跑；单 solo-maintainer 单源 |
| **I** | **Codex CLI 当日常工程系统（zh 实践者工作流）**（cnblogs 海军 8 个月，方法论回顾非单次可复现 run）<br>codex · **fail** ❌ · conf 0.7 · real ❌ | AGENTS.md + `~/.codex/config.toml` + Skills 三件套分层；复杂任务先 Plan 后实现；Playwright 视觉 diff 闭环（参考图 vs 实现迭代）；bounded-context 调试；`/review` 自审；MCP（Figma 设计转代码）扩展 | ✅ 三件套 + 视觉 diff 闭环 reverse-adapt 到 OpenClaw 几乎无改动 | ✅ **原生** | ✅ 配置/skill 概念对应 | 无独家短板。**整套 agent-agnostic**；短板是**方法论回顾、缺单次端到端 run 的命令/输出** |
| **J** | **Hermes 自学习预测市场天气交易 bot**（weatherbot repo 真实，但 Hermes 层+收益数字是营销）<br>other-frameworks · **fail** ❌ · conf 0.85 · real ❌ | 24/7 VPS 扫 Polymarket 天气市场，比 3 个预报源，EV+Kelly 定仓，交易低估温度桶，从自身结果写 skill 自校准 | OpenClaw 已有 Polymarket 工具，可加自学习层。但整体是**真钱交易 + astroturf 营销链 + 收益不可验证** | ⚠️ 可跑交易脚本 | repo README 根本未提 Hermes，自学习层是营销发明 | **拒**：真钱金融 + 收益数字不可验证 + 单源营销。与库内 polymarket-autopilot（仅模拟）重叠 |
| **K** | **Hermes Dreaming 空闲期记忆固化**（Minamaged18 plugin，但仿写自 OpenClaw 文档、无运行产物）<br>other-frameworks · **fail** ❌ · conf 0.85 · real ❌ | 空闲时 3 阶段（Light Sleep 暂存候选 → REM 写 DREAMS.md → Deep Sleep 评分晋升到 MEMORY.md），长期记忆自策展防腐烂 | **底层 OpenClaw Dreaming 才是真的**（memory-core 插件，Light/REM/Deep，6 权重信号）——Hermes plugin 反而是仿 OpenClaw 文档写的 | n/a | repo 仅 1 commit / 3★，无 DREAMS.md/MEMORY.md 样本，cron 机制还与营销 blurb 矛盾 | **反向**：这是 Hermes 仿 OpenClaw，不是 port 进 OpenClaw。OpenClaw 原生 Dreaming 已存在 |
| **L** | **Hermes Skill Factory 自动从工作流著作 skill**（Romanescu11 repo 349★，但 how-it-works 自承 plugin.py 含 TODO 未实现）<br>other-frameworks · **fail** ❌ · conf 0.83 · real ❌ | meta-skill 静默观察会话，当 agent 解决非平凡问题（5+ 工具调用）自动提议生成可复用 SKILL.md + plugin.py | OpenClaw/Claude Code skills 目录可接此 meta-skill。但**生成机制未实现（TODO），无端到端运行** | ⚠️ 同 | 原生 Hermes，但是 aspirational spec | **拒**：未实现的概念 + 单源 vendor 生态 + Hermes 产品本身星数不一致 |
| **M** | **CrewAI 结构化分歧 Crew（spec/test/coder/reviewer）**（PwC 案例 10%→70%，纯 vendor 营销无可复现内容）<br>other-frameworks · **fail** ❌ · **dup** ✅ · real ❌ | 角色化 crew 互相挑战产出，精度来自结构化分歧而非更大模型 | OpenClaw/Claude Code 子 agent 做质量门。**但已被 agent-swarm-dev-team 的三模型 PR 审查覆盖** | ⚠️ | ⚠️ | **近重复**：agent-swarm-dev-team 已实现「对抗式审查作质量门」；PwC 数字不可复现 + 专有语言 + 商业 SaaS（CONTRIBUTING 明确不正文推荐） |

---

## 2. 值得 port 进本库（作为 agent-agnostic 用例）的模式

按「screen 是否过 bar × 可复现性 × 与现有库的互补性」分三档。

### 第一档：直接立项（screen 已 passes_bar，证据硬，反向移植干净）

1. **B — 隔夜自愈自治循环（STATUS.md handoff + watchdog + fresh-context 顺序会话 + 输出落盘）** —— **最该优先 port**。
   - 理由：唯一一个反向适配「几乎一一映射 OpenClaw 原语」（cron + sessions_spawn fresh context + file handoff + watchdog 巡检）的候选；evekhm 有公开 repo + 可跑的 `test_goal.sh` demo（~2-3 分钟），读者真能照做。
   - 与现有库互补：`autonomous-project-management`（STATE.yaml 并行）和 `self-healing-home-server`（always-on 基础设施）都**没有**记录「长时无人值守如何不被上下文溢出搞死」这个具体失败模式 + 修复 kit。这是缺失的「如何熬过通宵」运维 playbook。
   - 诚实化要求：标题不要用「12h 跑通 5 阶段零失败」（作者自称 conceptual demo，phase prompt 是占位符），用「单工程师可复现的 fresh-context 长会话存活模式 + 可跑 demo」。

2. **G — OSS 维护即可复用 skills（AGENTS.md 强制触发 + GitHub Actions）** —— **强立项**。
   - 理由：唯一一个有**第一方组织级真实运营 + 活仓库可查 + 量化数据（+45% PR）**的 codex 候选；skill 概念三家通用，reverse-adapt 到 OpenClaw/Claude Code skills 几乎无改动。
   - 与现有库互补：全库无「维护者侧 / skills-as-CI 可复用工作流 / AGENTS.md 强制触发」模式；补齐「保持已发布 repo 健康」这条 build-side 编排之外的回路。
   - 诚实化要求：明确标注「OpenClaw 映射是 contributor 改写，OpenAI 自用的是 Codex，并未在 OpenClaw 上跑」。

3. **A — 单会话 Subagent 扇出做大规模迁移（Dynamic Workflows）** —— **立项，但定位为「Claude Code 原生旗舰 + 反向移植说明」**。
   - 理由：证据最硬（Bun PR #30412 真实合并，+1M 行，6755 commits，官方 blog 点名）；与库内 `agent-swarm-dev-team` / `multica-managed-agents`（fleet/团队模型）**正交**——这是「一个任务拆成几百个 ephemeral 文件级 subagent 单会话扇出 + 双 reviewer gate + checkpoint 续跑」的迁移模式。
   - 诚实化要求（screen 强调）：verified run 是 Claude Code native，OpenClaw/Hermes 反向适配**未被任何来源跑过**，必须写成「Claude Code 上已验证，反向适配到 OpenClaw/Hermes」而非声称可复现的 OpenClaw run；修正 404 源（用 PR #30412 替换已删分支）；~300 规则 PORTING.md 数字要 hedge；保留 high-token-cost / code-correctness-risk 标签。

### 第二档：值得 port，但需实质性重写才能过 bar

4. **横切 playbook（从 E + D 提炼）：「在 danger-full-access / 自治编码循环里安全跑 agent」的沙箱 + 安全 gate 防御纵深**。
   - 来源 E 的沙箱/egress/凭证隔离 playbook 是全库独一份的横切知识，但 E 本身闭源不可复现、且不是 OpenClaw 用例；D 的安全审查 gate 官方工具真实但反向适配未测试、且有「in-session 不 block / 只 CI Action 硬 gate」的事实纠错。
   - 建议：不要照搬任一候选，而是合成一个**用公开工具可复现**的 OpenClaw 用例——pre-merge 子 agent 安全审查（advisory）+ CI Action 硬 block + OpenClaw sandboxing/egress 配置，明确标 advisory-vs-blocking 边界，并配一个真实采用者（如 Deriv 风格）。这是 A/B 这类自治编码循环天然需要的安全护栏。

5. **F — 对抗式 bug hunter + 安全分支自动修（Hunter/Skeptic/Referee）** —— **缓议，待第三方运行证据**。
   - 模式 agent-agnostic 且新颖（多裁判压误报 + 安全分支 canary，库内无对应），但**零第三方端到端运行证据**（单作者 / 0 issue / 仅 star）。若出现真实实践者 write-up 再立项。

6. **I — Codex CLI 当日常工程系统（含 Playwright 视觉 diff 闭环 + AGENTS.md/config.toml/Skills 三件套）** —— **重写后可立项**，且**天然契合 zh 受众**（原文即中文）。
   - 库内代码用例全是团队/编排尺度，缺「单开发者如何日常活在 CLI 里」这一层。需重写成可复现配方：一个 runnable `config.toml` + AGENTS.md 模板 + 一个 Skill 骨架 + Playwright 视觉 diff 的实际命令，并端到端验证至少一个切片；加「中国用户适配 / 跨 Agent」note 映射到 Claude Code/OpenClaw。

### 第三档：不要 port

- **C / H**：概念无独家、原语三家都有，但唯一第一人称证据分别来自 AI 营销 persona（C）和私有 runtime 单源（H），不可复现 → 拒。
- **J（真钱金融 + astroturf 营销 + 收益不可验证，与 polymarket-autopilot 重叠）**、**K（Hermes 仿 OpenClaw 文档、无运行产物；且方向反了——OpenClaw 原生 Dreaming 已存在）**、**L（生成机制 TODO 未实现 + 单源 vendor 生态）**、**M（近重复 agent-swarm-dev-team + PwC 纯营销不可复现 + 商业 SaaS）** → 全部拒。

---

### 一句话结论

跨 agent 适配性最干净、最该立即 port 的是 **B（隔夜自愈循环）** 和 **G（维护即 skills）**——两者都 passes_bar、有可查证据、且反向映射到 OpenClaw 原语几乎无损；**A（单会话扇出迁移）** 作为「Claude Code 原生旗舰 + 诚实标注反向移植未跑过」收录。真正属于 Claude Code 独家、其他 agent 无干净对应的是 **A 的「单会话内写 JS 计划 → 几百 ephemeral subagent → 双 reviewer 逐文件 gate」** 这一 Dynamic Workflows 一等公民编排能力；而 **D 的 pre-write 硬安全 gate** 也无干净对应（官方 in-session 仅 advisory）。值得作为横切知识提炼的是 **E 的 danger-full-access 沙箱 playbook**，但须重写为公开可复现版本。

相关文件（均为本库现有、可作为互补/去重参照的绝对路径）：
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/agent-swarm-dev-team.md
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/multica-managed-agents.md
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/autonomous-project-management.md
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/self-healing-home-server.md
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/project-state-management.md
- /Volumes/WDBlack/dev/Github/awesome-openclaw-usecases-zh/usecases/polymarket-autopilot.md
