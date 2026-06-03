<!-- 本文件由多 Agent 研究工作流生成（2026-06），经对抗式核验与去重。候选用例均未由本仓库维护者亲自跑通，**不构成收录**——按 CONTRIBUTING 的“真实跑通 + 多源验证 + 可复现”红线，需先验证一次真实运行后才可成稿入库。 -->

# Candidate Shortlist — Multi-Agent Usecase Library Expansion

**Scope:** 21 screened survivors across 11 lanes. Tier A = `real_user_confirmed=true` AND `confidence>=0.6`. Tier B = passed bar but `real_user_confirmed=false`. Grouped by vertical. Evidence caveats are surfaced, not buried.

**Existing-50 coverage (for under-served analysis):** dev/agent-orchestration, CN-platform assistants, market-monitoring finance, research/paper digests, personal productivity, content production, channel-integration customer service, observability, home-server infra. **No existing case** touches: legal, professional healthcare, accounting/bookkeeping, HR/people-ops, real estate, marketing/SEO, education, ecommerce write-ops, or agent security.

---

## TIER A — High-confidence, real-user-verified, ready to draft

### Legal (UNDER-SERVED → currently 0 cases)

**A1. Claude for Legal — official Anthropic plugin marketplace** · `legal/compliance` · target: Claude Code (+ Cowork, Managed Agents API)
What: Anthropic's official open-source plugin suite — 12 practice-area plugins, ~80-90+ slash-command agents (`/commercial-legal:review`, `/privacy-legal:dsar-response`, `/corporate-legal:tabular-review`), cold-start interview writes a CLAUDE.md practice profile, MCP-grounded citations.
Best sources: `github.com/anthropics/claude-for-legal` · `artificiallawyer.com/2026/06/01/claude-for-legal-has-over-90-ai-agents/`
Risk tags: external-api, credential-heavy, privacy, professional-liability
Why complementary: Anchor legal case; vertical-professional application of the library's managed-agent/orchestration direction. Distinct = plugin-marketplace + practice-profile + MCP-grounded.
**Evidence gap (must surface in draft):** the ONE logged practitioner run (Ahmad Gado, Kallam AI — a competitor with disclosure) was via **Cowork** (manual zip-upload), NOT the headline Claude Code `/plugin` flow; NDA/DSAR/claim-chart agents are documented-capability + press only, no logged run. Fix specs: "12 plugins / 80-90+ agents" (not 13/70+); drop unconfirmed TechCrunch/Fortune cites.

**A2. Grounded legal research via CourtListener MCP** · `legal/research` · target: Claude Code, Claude Desktop, any
What: Free Law Project's official MCP connector gives an agent citation-verified access to US case law, PACER dockets, citation networks — primary data, anti-hallucination cite-checking.
Best sources: `free.law/2026/05/12/courtlistener-is-now-available-inside-claude/` · `courtlistener.com/help/mcp/`
Risk tags: external-api, read-only
Why complementary: The "grounded research / paralegal" pillar; a free, authoritative legal-data building block other cases can cite. Verified-primary-source pattern vs the library's web/social aggregation.
**Evidence gap:** the proof-of-pattern run (AI Law Librarians, 2026-04-22, caught fabricated quotes/inverted holdings) used a **community-built self-hosted** CourtListener MCP; no logged session on the official `mcp.courtlistener.com` connector specifically. Same data, same pattern, different implementation.

### Finance / Accounting (UNDER-SERVED → existing finance is market-monitoring only; 0 bookkeeping/AP cases)

**A3. Plain-Text Accounting in Obsidian with hledger + Claude Code** · `personal-finance/bookkeeping` · target: Claude Code, any · **strongest real-user signal of the finance lane**
What: Double-entry hledger journals in an Obsidian vault; Claude Code imports CSVs, tags transactions by cross-referencing vault notes, runs weekly reports via headless `claude -p` cron.
Best sources: `mandalivia.com/obsidian/hledger-obsidian-personal-finance-with-claude-code/` · `hledger.org/obsidian.html` (official docs list it)
Risk tags: writes-local, privacy, financial
Why complementary: Fills the bookkeeping gap; fully local (no API/credentials); reuses the Obsidian/second-brain theme without duplicating second-brain.md.
**Evidence gap (minor):** single primary practitioner; the hledger maintainer (Simon Michael) shared it but did not independently re-run. Repo is OpenClaw-terminology-first → writeup must map `claude -p`/cron to OpenClaw Cron via CONCEPT-MAPPING.

**A4. Company Credit-Card Expense Reconciliation (receipts + calendar matching)** · `expense-reconciliation/AP` · target: Claude Code, any
What: Cut monthly card-expense reporting from ~2h to minutes — upload statement CSV + calendar export, Claude writes an ICS→CSV preprocessing script, then drafts business-purpose explanations by matching charges to calendar events.
Best sources: `cirit.fi/blog/streamlining-credit-card-reports-with-claudeai` (Harri Lammi, verified real person, first-person run)
Risk tags: financial, privacy, writes-local
Why complementary: AR/AP slice the library lacks; distinct from hledger (business expense reporting vs personal double-entry). Honest semi-automation framing.
**Evidence gap:** single personal/company blog (not independently reproduced); the Anthropic corroboration source is promotional template doc, not a user transcript; First Card is Finland-specific (pattern generalizes).

**A5. QuickBooks / TurboTax inside Claude (Intuit-Anthropic connectors)** · `smb-accounting/tax` · target: Cowork, Small Business, Claude.ai, Claude Code, Enterprise
What: Official Intuit MCP — Claude reads/writes QuickBooks (categorization, reconciliation, P&L/cash-flow, CPA exports) plus TurboTax tax Q&A.
Best sources: Intuit investor press release (Feb 2026) · `intuit.com/blog/news-social/intuit-apps-now-available-in-claude/` · QuickBooks Connector help article
Risk tags: financial, external-write, external-api, credential-heavy
Why complementary: SMB SaaS bookkeeping + consumer tax — a segment nothing in the library touches.
**Evidence gap (significant — lowest-priority of the finance group):** the cleanest first-person run (Alex Altman, Substack, ~60k GL rows consolidated) used the **direct QuickBooks Developer API + Claude Code, NOT the official connector** (which didn't exist yet). Connector-path real-user reports are thin (short X posts: 3-6 min reconciliation, OAuth gotcha) + one YouTube. US-centric, SaaS/credential-heavy, write-back to live financial data — weakest CN fit. Best framed as emerging.

### E-commerce ops (write-layer) (UNDER-SERVED → existing cn-ecommerce is read-only alerts on mock data)

**A6. Daily Amazon Ops Autopilot: Inventory Reconciliation + Reorder + Review Requests** · `marketplace-ops/Amazon FBA` · target: Claude Code, OpenClaw, any · **documented dollar ROI**
What: Claude Code handles recurring FBA chores — flags sent-vs-received mismatches on Closed shipments, queues reimbursement cases, batch-sends review requests via SP-API, answers "what to reorder." Evolves /dailyupdates slash command → GitHub Actions cron + email alerts.
Best sources: `theautomatedoperator.substack.com/p/handing-my-daily-tasks-off-to-claude` (Alex Willen, first-person, $1,411 reimbursement screenshot) · `buildtolaunch.substack.com` interview (~30% ops automated)
Risk tags: external-api, external-write, credential-heavy, financial
Why complementary: Adds the operational write/loop layer on a REAL account that cn-ecommerce-multi-agent (read-only mock-data alerts) lacks.
**Evidence gap (minor):** the reorder/velocity claim is real but **mis-cited** — it's in a different post by the same author ("Cashflow Projection Dashboard"); add it to sources. Writeup must add write-op human-confirmation gates per repo convention.

### Marketing / SEO (UNDER-SERVED → 0 cases; only a "marketing agent" inside multi-agent-team)

**A7. Corey Haines marketingskills library install** · `marketing-seo` · target: Claude Code, Codex, any
What: One install drops a 30+ skill marketing pack (CRO, copy, SEO, ad-creative, content-calendar, churn) into any agent, turning it into a marketing operator; 80+ tool registry (Ahrefs/GA4/Meta/Stripe via CLIs/MCP/Composio).
Best sources: `github.com/coreyhaines31/marketingskills` (~31.7k stars, v2.3.0) · `composio.dev/content/best-marketing-skills`
Risk tags: external-api, credential-heavy, external-write
Why complementary: The "equip any agent for marketing" entry point — agent-agnostic skill-pack-install pattern the library lacks.
**Evidence gap:** real end-to-end use shown via demos (Corey + third party Andrew; @theaiimpact walkthrough) but **no verified business-OUTCOME metrics**. Fix specs: "80+ tools registry + CLIs/MCP/Composio" (not "50+ bundled"); clarify skills are markdown directives pointing to a separate tools layer (not zero-config); live data needs credential/MCP setup.

**A8. Get Cited by AI Search (GEO/AEO)** · `marketing-seo/content` · target: Claude Code, Codex, any
What: Agent monitors brand citations across ChatGPT/Perplexity/Gemini/Claude/AI-Overviews, scores content "citability," drafts/ships content to close gaps. New 2026 discipline (optimize to be quoted, not ranked).
Best sources: `dev.to/whollykaw/i-built-an-ai-powered-seo-stack-with-claude-code...` (21 mentions / 758K impressions, 3 mo) · `github.com/AgriciDaniel/claude-seo` (8k stars)
Risk tags: external-api, public-post, writes-local
Why complementary: Brand-new discipline absent from library; distinct from competitive-intelligence (which tracks competitors' pricing).
**Evidence gap:** the citability-audit/rewrite half is fully turnkey/reproducible; the cross-platform mention-TRACKING half needs a **paid API (DataForSEO ~$150/mo) with no reader-copyable script**, and the full "track→draft→ship" loop is assembled from multiple sources, not one shipped pipeline by one user. Writeup should split into (a) free citability-audit and (b) optional paid tracking add-on; adapt for CN (Baidu/Doubao/Kimi).

### DevOps (partially served → CI-triage & dependency-upgrade are gaps)

**A9. CI Babysitter: auto-triage GitHub Actions failures + push fix PRs** · `devops/ci-cd` · target: Claude Code, OpenClaw, Codex, any
What: Agent watches a PR's Actions runs on a loop; on red it reads logs via `gh`, finds root cause, edits/pushes/re-runs until green.
Best sources: `neonwatty.com/posts/claude-code-ci-babysitter/` (Jeremy Watt, run #139 red → #140 green, concrete artifacts) · `github.com/anthropics/claude-code-action`
Risk tags: writes-local, external-write, external-api
Why complementary: Tightly-scoped single-repo CI-triage unit of work, distinct from orchestration-heavy agent-swarm-dev-team.
**Evidence gap / REQUIRED FIX:** the candidate over-cites — dev.to/kochan explicitly says it "stays at investigation level / NOT yet implemented," so it is endorsement-only, NOT corroboration. Replace with trashhalo gist / community skills. Add 人机分工 + external-write confirmation gate for remote push.

**A10. Dependency Upgrade Agent: bump major version + auto-fix breaking changes** · `devops/code-maintenance` · target: Claude Code, Codex, OpenClaw
What: Point agent at a dep to bump; it reads the changelog, plan-modes the breaking changes, edits all call sites, runs tests, iterates to green.
Best sources: **TalentLMS Tech Blog — Alexander Antoniades, React 18→19 (forwardRef removal, types-react-codemod, Betterer)** — surfaced by the screener, NOT in the candidate's original URLs · `recombobulate.dev/tips/...` (recipe, supporting only)
Risk tags: writes-local, external-api
Why complementary: In-language major-version upgrade + test-driven repair; nothing in the library covers dependency upgrades.
**Evidence gap / REQUIRED FIX:** the candidate's two cited non-doc sources are weak (a generic recipe + an AutoGPT *review* issue = demand signal only). Anchor the writeup on the TalentLMS run. The candidate's claim that a "cross-language migration case exists in the library to complement" is **false** (grep false positives) — correct that framing.

### Research / Data (partially served → SLR/reproducibility are gaps)

**A11. Econometrics Replication Package Auditor (Claude Code + R, AEA-compliant)** · `academic/reproducibility` · target: Claude Code
What: Academic harness (38 skills, 18 agents) runs R analysis then audits the manuscript against the code — `/audit-reproducibility` enforces numeric-tolerance thresholds, writes a per-paper passport.yaml (PASS/FAIL/STALE/UNVERIFIED), assembles AEA replication packages.
Best sources: `github.com/pedrohcgs/claude-code-my-workflow` (Pedro Sant'Anna, Emory econ prof, JoE AE; v1.9.0) · Scott Cunningham (Baylor) X post running an end-to-end audit of 6 Callaway-Sant'Anna packages
Risk tags: writes-local, external-api
Why complementary: Reproducibility/replication angle absent from library; the only academic case (arxiv-paper-reader) is read+compile-LaTeX.
**Evidence gap (minor):** repo ships no bundled worked example / sample passport.yaml — a reader must supply their own R analysis + manuscript. Cite Cunningham's run as proof-of-use, SKILL.md as runnable spec.

**A12. Agentic Paper Replication Bench (read paper → rewrite code → verify vs published)** · `meta-science/reproducibility` · target: Claude Code, Codex
What: Agent extracts a paper's methods (masking results), reimplements from scratch, runs on original data, compares coefficient signs / 95% CIs cell-by-cell, diagnoses discrepancies — a referee/journal-replication check.
Best sources: arXiv 2604.21965 "Read the Paper, Write the Code" (Kohler et al., ETH/Basel; **public runnable repo** `github.com/benjamin-kohler/social_science_replicability`) · arXiv 2602.16733 "Scaling Reproducibility" (Xu, Stanford; Yang, HKBU)
Risk tags: writes-local, external-api
Why complementary: Reproduces SOMEONE ELSE'S paper from text+data (vs A11 auditing your own); two independent credentialed academic teams ran it end-to-end on Claude Code.
**Evidence gap / REQUIRED FIX:** source-2 stats in the candidate ("87% / 92 IV papers / 215 specs") **don't match** what was verified (82% on 67-paper sample, 100% on 25 new; broader: 384 studies / 3,382 models) — fix the numbers. Reproducibility-by-reader is the weak leg (research-grade, multiple CLIs + API keys + datasets, difficulty=hard) — must be trimmed to a follow-along workflow.

### Customer Support (partially served → source-grounded answers & KB-from-tickets are gaps)

**A13. Codebase-Grounded Support Engine + Slack-to-KB Pipeline (Al Chen / Galileo)** · `B2B SaaS support / source-grounded RAG` · target: OpenClaw, Claude Code, Codex
What: Field engineer points Claude Code at all 15 product repos at once (multi-repo workspace + a Claude-written pull_all script refreshing main branches daily) + Confluence per-customer quirks, then turns resolved Slack threads into KB articles.
Best sources: Lenny Rachitsky "How I AI" (aired 2026-04-06) · `chatprd.ai/how-i-ai/claude-code-and-repos-to-answer-any-customer-question`
Risk tags: external-api, credential-heavy, writes-local, privacy
Why complementary: Inverse of multi-channel-customer-service (live-chat from static FAQ) — answers hard technical tickets via live codebase + auto-generates KB from tickets.
**Evidence gap / WRITING CONSTRAINT:** the KB-from-tickets step leans on Pylon's proprietary one-click feature → per repo rule ("商业 SaaS 产品不在正文推荐") reframe as a generic agent flow (fetch thread → draft → human review → publish), Pylon as optional reference.

### HR / People-Ops (UNDER-SERVED → 0 cases)

**A14. Anthropic Official HR Plugin: People-Ops Lifecycle** · `hr/people-ops` · target: Claude Code, Cowork, any
What: Official HR plugin (9 skills: recruiting-pipeline, interview-prep, draft-offer, onboarding, policy-lookup, comp-analysis, performance-review, org-planning, people-report) on Claude Code/Cowork; standalone or via ATS/HRIS/Slack MCP.
Best sources: `github.com/anthropics/knowledge-work-plugins/tree/main/human-resources` · AIHR (Erik van Vulpen) ran /comp-analysis end-to-end vs Ravio market data, published hard metrics
Risk tags: external-api, external-write, privacy, credential-heavy
Why complementary: Net-new HR vertical; official-plugin install pattern parallels multica-managed-agents. Pairs with existing lark-* skills for a CN stack.
**Evidence gap:** the ONE documented run covers comp-analysis only (1 of 9 skills), which AIHR found **unreliable for senior roles** (50-80% under-estimated) — must caveat and require external validation.

### Real Estate (UNDER-SERVED → 0 cases)

**A15. CRE Lease Management Agent (VP Real Estate plugin)** · `commercial real estate / leasing` · target: Claude Code
What: Plugin marketplace digitizing a 20-yr CRE veteran's workflow — abstracts lease PDFs, effective-rent analysis, tenant-credit scoring (15+ ratios), rollover-risk, IFRS 16 / ASC 842 schedules; 23 slash commands, 11 calculators.
Best sources: `github.com/reggiechan74/vp-real-estate` (Reggie Chan CFA FRICS, his own production tool; v3.0.0, ships sample inputs+outputs) · `linkedin.com/in/reggiechan`
Risk tags: writes-local (drop "credential-heavy" — no credentials needed)
Why complementary: Zero real-estate cases; genuine professional depth (effective rent, tenant credit, IFRS 16). Practitioner shipping his own daily tool = strongest authentic signal.
**Evidence gap / REQUIRED FIX:** README install path reads `reggiechan/...` but working marketplace path is **`reggiechan74/vp-real-estate`** — use the 74 form or install fails. Third-party run reports are thin (author + repo artifacts + LinkedIn only).

### Education (UNDER-SERVED → 0 cases)

**A16. codebase-to-course: turn any repo into an interactive HTML course** · `education / dev-onboarding` · target: Claude Code
What: Skill points at a repo, generates a self-contained single-page HTML course (scroll modules, architecture diagrams, code translations, quizzes, glossary) for vibe coders / new hires.
Best sources: `github.com/zarazhangrui/codebase-to-course` (~4.5k stars) · XDA Developers (independent writer ran it on their 800-line Java project, flagged hardcoded creds)
Risk tags: writes-local (external-api debatable — only uses the running model)
Why complementary: No education/course-building case exists; generates a pedagogical artifact (vs retrieval in knowledge-base-rag). Strong CN fit (onboarding juniors, course material from internal repos).
**Evidence gap:** clean — independent third-party run + awesome-claude-code acceptance. Only the X view-count metric is unverifiable (not load-bearing).

### Flagship Claude Code / Codex patterns (partially served → these are distinct axes)

**A17. Single-Session Subagent Fan-Out for Massive Migration (Dynamic Workflows)** · `large-scale code migration` · target: OpenClaw, Hermes, Codex
What: Orchestrator writes a JS plan, fans ONE migration out to tens-hundreds of parallel ephemeral subagents in one session (map lifetimes → port file-by-file → two reviewers/file → build-test fix-loop), checkpointed/resumable.
Best sources: `claude.com/blog/introducing-dynamic-workflows-in-claude-code` · **Bun PR #30412** "Rewrite Bun in Rust" (Jarred Sumner, merged 2026-05-14, +1,009,257 LOC / 6,755 commits, flagship example)
Risk tags: writes-local, external-api, high-token-cost, code-correctness-risk
Why complementary: Orthogonal to agent-swarm-dev-team/multica (fleet/team patterns) — this is single-session fan-out for one task.
**Evidence gap / REQUIRED FIX:** source URL `.../compare/claude/phase-a-port` is now **404** (branch deleted) → replace with PR #30412; the "~300-rule porting doc" figure is soft (PORTING.md gone) → hedge. OpenClaw/Hermes reverse-adaptation is **plausible but NOT demonstrated** — write honestly as "verified on Claude Code, reverse-adapted."

**A18. Overnight Self-Healing Autonomous Loop (STATUS.md handoff + watchdog + headless --print)** · `autonomous long-running ops` · target: OpenClaw, Hermes
What: Bash orchestrator launches sequential fresh headless `claude --print` sessions, each reading/updating a STATUS.md handoff (no carried context), with phased /goal conditions + a watchdog that restarts from the next checkpoint; CLAUDE.md rules force output-to-file to beat context overflow.
Best sources: `medium.com/@evekhm/running-claude-code-autonomously-overnight...` · `github.com/evekhm/claude-code-blog/tree/main/long-sessions` (runnable scripts + e2e tests, inspected)
Risk tags: writes-local, external-api, dangerously-skip-permissions, unattended-autonomy
Why complementary: Documents the specific failure mode (compaction thrashing over long runs) + fix kit absent from autonomous-project-management / self-healing-home-server.
**Evidence gap (notable):** the "ran 12h, all 5 phases, zero failures" headline **overstates verifiable evidence** — author calls it "a conceptual demonstration, not production-grade"; run_autonomous.sh ships PLACEHOLDER prompts; only the short demo tests are end-to-end-tested. **Multi-source is WEAK** (single author, 0 stars, article+own repo ≈ one source). Frame as a single-engineer reproducible pattern + demo, lead with test_goal.sh.

**A19. OSS maintenance as repeatable skills (AGENTS.md + GitHub Actions)** · `OSS repo maintenance / CI` · target: Codex, Claude Code, any
What: Codify recurring chores as repo-local skills in .agents/skills/ ($code-change-verification, $docs-sync, $final-release-review, $test-coverage-improver, $pr-draft-summary) with mandatory AGENTS.md triggers, invoked from GitHub Actions.
Best sources: `developers.openai.com/blog/skills-agents-sdk` (Kazuhiro Sera; OpenAI maintains its own Agents SDK repos this way — 457 PRs merged, +45%) · live `.agents/skills/` dir in `openai/openai-agents-python`
Risk tags: writes-local, external-api, external-write
Why complementary: Maintainer-side skills-as-CI / AGENTS.md mandatory-trigger pattern — complements the build-side cases (agent-swarm-dev-team).
**Evidence gap (minor):** demonstrated practice is Codex-centric (OpenAI's own tooling) — the OpenClaw/Claude Code framing is a contributor reverse-adaptation; must not imply OpenAI used OpenClaw.

### Smart Home / IoT (UNDER-SERVED → 0 device-control cases)

**A20. OpenClaw + Home Assistant: Natural-Language Smart Home Agent** · `smart home / IoT` · target: OpenClaw
What: Bridge OpenClaw to Home Assistant via ha-mcp so a persistent agent controls + repairs automations by NL/voice — diagnose a broken motion-light blueprint, rebuild it, speak through smart speakers.
Best sources: `dan-malone.com/blog/openclaw-home-assistant` (Dan Malone, first-person operated build, concrete Proxmox topology, diagnosed 2 dead Hue bulbs + rebuilt automation) · `eastondev.com/blog/en/posts/ai/20260205-openclaw-homeassistant/`
Risk tags: external-write, credential-heavy, privacy
Why complementary: Smart-home/IoT is a complete gap (closest = family-calendar, no device control). Showcases OpenClaw's "AI that acts."
**Evidence gap:** the hero "diagnose + rebuild" run is **single-source** (Dan Malone); setup mechanics are multi-source. Surface physical-device-write + long-lived-token risks; recommend scoped least-privilege token.

### Private/local analytics (UNDER-SERVED → all existing research is cloud-API)

**A21. Fully-Local Data Analyst (OpenClaw + Ollama, zero cloud)** · `private data analysis / on-device` · target: OpenClaw
What: Upload CSV to a small web UI → slash command → OpenClaw routes to local Python/Pandas + local Ollama (qwen3); emits trend PNG + Markdown report + JSON trace, nothing leaves the machine.
Best sources: `datacamp.com/tutorial/openclaw-ollama-tutorial` (Aashi Dutt, GDE ML/GenAI) · companion repo `github.com/AashiDutt/OpenClaw_Ollama` (load-bearing reproducibility source)
Risk tags: writes-local
Why complementary: No fully-local zero-cloud analytics pipeline exists; fits PIPL/data-sovereignty for CN audience.
**Evidence gap:** **single-first-party-runner** — authored how-to + same author's repo (~2 stars, 1 commit); no independent second runner; 2 of 3 source URLs are provider docs. Frame as an author-demonstrated reproducible pattern, NOT a widely-adopted workflow; cite the GitHub repo not just DataCamp.

---

## TIER B — Promising, but the user must verify a real run (`real_user_confirmed=false`)

### Legal / Compliance

**B1. GRC / compliance-gap-analysis skills (30 frameworks)** · `legal/compliance/security` · target: Claude Code, any · screen-confidence 0.70
30 installable Claude Code plugins (ISO 27001, SOC 2, GDPR, EU AI Act, HIPAA, DORA…), auto-activating for control mapping / gap analysis. `github.com/Sushegaad/Claude-Skills-Governance-Risk-and-Compliance` (~511 stars, MIT). Risk: read-only, writes-local, professional-liability.
**Why not Tier A:** Real named GRC pros tested it (Jaana Metsamaa, Shubham Mishra) but **no single documented end-to-end run** (install → gap analysis → audit-ready artifact → outcome). The 96%/81% eval is author-self-reported (no reproducible harness). Skill depth uneven across frameworks. Writeup: lead with the Claude Code marketplace path (resolves outdated web-only caveat); frame eval as author-reported.

### Healthcare — professional (UNDER-SERVED → only health-symptom-tracker, a consumer diary)

**B2. Anthropic Official Healthcare Marketplace: Prior-Auth + FHIR + Clinical-Trial** · target: Claude Code · screen-confidence 0.82
Official `anthropics/healthcare`; agent digests a prior-auth packet, validates NPI/ICD-10/CPT, cross-refs CMS rules, drafts medical-necessity summary. **Ships sample synthetic PDFs** + credential-free remote MCP → reproducible by any reader. `github.com/anthropics/healthcare`. Risk: external-api, privacy (note: "credential-heavy" is overstated — MCP needs no keys).
**Why not Tier A:** named customers (Banner, Commure, Stanford) are **organizational endorsements of Claude-for-healthcare broadly**, not runs of this repo; prior-auth self-labels as a "demo… customize." Writeup: enforce DRAFT-ONLY/human-review, use bundled samples (no real PHI), don't overclaim customers ran this exact repo.

**B3. PubMed-Driven Systematic Literature Review (PICO/PRISMA)** · target: Claude Code, Codex, OpenClaw · screen-confidence 0.70
PubMed MCP + literature-review skill: PICO question, title→abstract→full-text screen, PRISMA flow, Cochrane/NOS/AMSTAR-2, cited export. Multiple maintained repos (cyanheads/pubmed-mcp-server; davila7 skill in a 27.7k-star repo). Risk: external-api, read-only, writes-local.
**Why not Tier A:** Aaron Tay (SMU evidence-synthesis figure) runs **only search + citation-retrieval** with screenshots ("scratching the surface"); **no one published the FULL PRISMA-dedup → appraisal → cited-PDF pipeline.** Scope the writeup to what's demonstrated (search/retrieve/dedup/cite-check); label appraisal+export as documented-but-unverified.

**B4. Clinic Data Back-Office: Natural-Language FHIR/EHR Queries via MCP** · target: Claude Code, OpenClaw, any · screen-confidence 0.72
Connect Claude to a FHIR R4 server (Medplum/HAPI) via FHIR MCP for plain-language patient/condition/observation queries + synthetic test data. Build around the more-mature WSO2 repo (122 stars, v0.10.0, no-PHI docker-compose path) + Anthropic's official fhir-developer skill. Risk: external-api, privacy, credential-heavy.
**Why not Tier A:** the primary repo's only confirmed runs are **vendor self-demos** (Momentum/Bartosz Michalak); no independent third-party clinic deployment. Write around the reproducible WSO2 + public-sandbox path, strict read/structured-ops only.

### Research / Data

**B5. PRISMA Systematic Literature Review Agent (citation-contamination check)** · target: Claude Code, Codex · screen-confidence 0.72
Deep-Research "systematic-review" mode: multi-source discovery (Semantic Scholar/OpenAlex/Crossref/arXiv), two-pass screen, risk-of-bias, citation-verification across 3 indexes. `github.com/Imbad0202/academic-research-skills` (~26.6k stars, v3.10.0). Risk: external-api, writes-local.
**Why not Tier A:** massive adoption is **proxy only** — discussions are install errors / feature requests; "guide" articles are SEO README-rephrasing; **no independent first-person end-to-end run.** Author the usecase from a self-run; spell out the exact systematic-review trigger prompt; note ~$4-6 API budget. (Note: overlaps B3 thematically — pick one PRISMA case or differentiate clearly.)

**B6. CSV-to-Deck Data Analyst Swarm (frame→analyze→story→slides)** · target: Claude Code · screen-confidence 0.62 (**lowest — borderline**)
18-agent/4-phase DAG: connect to CSV/DuckDB/Postgres/BigQuery/Snowflake, EDA + root-cause with source tie-out, auto-build Marp deck. `github.com/ai-analyst-lab/ai-analyst` (238 stars, 606 tests). Risk: writes-local, external-api.
**Why not Tier A:** **no independent third party ran THIS toolkit** — the practitioner articles use generic Claude Code; the repo is a **paid-course companion** (Maven bootcamp, sole contributor), 238 stars but 0 issues in 3.5 mo → presenting stars as organic adoption would be source-laundering. Only publish if framed as "course-companion toolkit demonstrating the pattern," with an editor actually running /run-pipeline first.

### DevOps / SRE (UNDER-SERVED → no alert-driven incident triage)

**B7. SRE On-Call Incident Responder: PagerDuty → root cause → approval-gated fix PR** · target: Claude Code, any · screen-confidence 0.74
Anthropic Managed-Agents cookbook: ingest PagerDuty alert, consult runbooks, read logs/k8s in a sandbox, open minimal-diff PR, merge only after human approval (Slack button). `platform.claude.com/cookbook/managed-agents-sre-incident-responder` (executed notebook with output cells). Risk: external-api, external-write, credential-heavy, writes-local. Difficulty: hard.
**Why not Tier A:** **every concrete account traces to the SAME Anthropic cookbook** — third-party pieces are reviews/overviews, not independent production runs (vendor-run, not user-confirmed). Required disclosures: Managed Agents is a **gated BETA**; the notebook runs against **MOCKED** PagerDuty/GitHub/Datadog fixtures (production swaps sketched, not executed); the PagerDuty plugin is early/low-adoption. Frame as "Anthropic-official reproducible tutorial," not "battle-tested."

### Customer Support

**B8. Official Anthropic Customer-Support Plugin (/triage /draft-response /escalate /kb-article)** · target: Claude Code, Cowork, any · screen-confidence 0.74
Official customer-support plugin (5 commands, 5 skills: P1-P4 taxonomy, SLA tiers, tone, article standards) + Slack/Intercom/HubSpot/Guru MCP. `github.com/anthropics/knowledge-work-plugins/tree/main/customer-support` (~19k stars). Risk: external-api, external-write, credential-heavy, privacy.
**Why not Tier A:** reviewers dissect/confirm-it-installs but **none ran it on a real ticket queue with metrics**; Anthropic's 50.8% support stat is from Intercom/Fin, NOT this plugin. Frame outcomes as "capability, not measured result"; CN adaptation (Intercom/Guru → Lark/WeCom + Lark Wiki).

### Agent Security (UNDER-SERVED → 0 cases)

**B9. OpenClaw Self-Defense: Skill Supply-Chain & Threat Monitor** · target: OpenClaw, Hermes · screen-confidence 0.68 (**lowest in tier**)
Security skill scanning an OpenClaw deployment for compromised skills/IOCs (ClawHavoc, AMOS/Vidar, reverse shells, 60+ CVEs); auto-updating IOC DB, local dashboard, remediation, cron self-audit. `github.com/adibirzu/openclaw-security-monitor` (MIT, v5.3.2) + corroborating prompt-security/clawsec, SlowMist guide. Risk: credential-heavy, writes-local.
**Why not Tier A:** the only repo issue is a feature request from a non-user; **no end-user "I ran this and it caught X" writeup.** Evidence = maintainer + modest adoption (45 stars). Highly topical (real 2026 supply-chain attacks); accept as a maintainer-grade runnable skill, do not claim a named end-user; cite clawsec/SlowMist as broader multi-team signal.

---

## Vertical coverage summary

| Vertical | Existing-50 status | Shortlist adds |
|---|---|---|
| **Legal** | UNDER-SERVED (0) | A1, A2, B1 |
| **Healthcare (professional)** | UNDER-SERVED (only a consumer diary) | B2, B3, B4 |
| **Accounting / bookkeeping / AP** | UNDER-SERVED (finance = market-monitoring only) | A3, A4, A5 |
| **HR / people-ops** | UNDER-SERVED (0) | A14 |
| **Real estate** | UNDER-SERVED (0) | A15 |
| **Marketing / SEO** | UNDER-SERVED (0) | A7, A8 |
| **Education** | UNDER-SERVED (0) | A16 |
| **Agent security** | UNDER-SERVED (0) | B9 |
| **E-commerce write-ops** | partial (read-only mock alerts only) | A6 |
| **DevOps / SRE** | partial (build/infra, no CI-triage/dep-upgrade/incident) | A9, A10, B7 |
| **Research / reproducibility / SLR** | partial (paper-reading only) | A11, A12, B3, B5, B6 |
| **Customer support** | partial (channel-integration only) | A13, B8 |
| **Smart home / IoT** | UNDER-SERVED (0 device-control) | A20 |
| **Private/local analytics** | UNDER-SERVED (all research is cloud) | A21 |
| **Flagship agent patterns** | partial (fleet/team only) | A17, A18, A19 |

**Most acute gaps to prioritize:** Legal, professional Healthcare, Accounting, HR, Real estate, Marketing/SEO, Education, and Agent security each have ZERO existing coverage — these expansions move the library decisively "beyond OpenClaw" into vertical-professional territory.

## Honest evidence caveats (do not inflate)

- **Strongest real-user proof (single named practitioner, full run, concrete artifacts):** A3 (hledger), A4 (expense recon), A6 (Amazon FBA, $ ROI), A9 (CI babysitter), A11 (Cunningham audit), A13 (Al Chen), A15 (Reggie Chan's own daily tool), A16 (XDA writer), A17 (Bun PR), A19 (OpenAI's own repos). These are the safest to draft.
- **"Verified pattern, but the logged run used a different implementation/path than the headline":** A1 (Cowork not Claude Code; competitor reviewer), A2 (self-hosted not official connector), A5 (direct API not official connector).
- **Single-source / single-author (real but thin):** A18 (1 author, 0 stars, conceptual demo overstated), A20 (hero run single-source), A21 (1 author repo, 2 stars).
- **All Tier B require the user to verify a real run before drafting as "confirmed."** Highest-risk Tier B: B6 (course-companion stars ≠ adoption; needs an editor to actually run it) and B7 (every account = the same vendor cookbook, gated beta, mocked fixtures).
- **Number/spec fixes required before publishing:** A1 (12 plugins/80-90+ agents), A7 (80+ tools), A12 (correct the 87%/92/215 figures), A17 (replace 404 URL with PR #30412, hedge "~300 rules"), A6 (add the mis-cited reorder post), A15 (use the `reggiechan74` install path, drop credential-heavy), A9/A10 (replace weak corroboration sources flagged by the screener).
- **Potential internal overlap to resolve:** B3 and B5 are both PRISMA SLR cases (PubMed-specific vs multi-source) — differentiate or pick one.
