# 求职 Agent 团队（全流程自动化）

求职是一项系统工程——搜索岗位、定制简历、准备面试、谈薪策略，每个环节都需要不同的专业知识。手动操作时，求职者常常在这些角色之间反复切换，效率低下且容易遗漏。如果你能启动一个 4 Agent 求职团队，让每个智能体专注一项能力，从发现岗位到拿到 offer 全程协作呢？

这个用例基于 CrewAI + Claude 的多智能体架构，将求职流程拆分为 4 个专业角色：岗位搜索、技能顾问、面试教练、职业策略师。每个 Agent 顺序执行并传递上下文，最终输出一份涵盖岗位分析、学习路线、面试题库和求职策略的完整报告。

## 痛点

- **简历千篇一律**：同一份简历海投几十家，ATS（Applicant Tracking System，简历追踪系统）关键词匹配率低，大量投递石沉大海
- **技能差距看不清**：岗位描述罗列了 20 项要求，分不清哪些是硬性门槛、哪些是加分项，准备方向模糊
- **面试准备靠运气**：网上搜的通用题目和实际岗位要求脱节，技术面、行为面、情景面各需不同准备方法
- **求职策略缺失**：不知道什么时候投递最佳、要不要写 Cover Letter（求职信）、如何跟进、怎么谈薪
- **时间精力耗尽**：每个岗位从研究到投递平均花 2 小时，同时找 20 个岗位就是一周的全职工作量

## 功能介绍

- **4 Agent 流水线**：Job Searcher → Skills Advisor → Interview Coach → Career Advisor，上一步输出自动成为下一步的上下文
- **实时岗位搜索**：通过 Adzuna API 拉取真实在招岗位，按关键词、地区、远程状态筛选
- **加权技能分析**：从多条岗位描述中提取技能词频，按 Critical / Important / Nice-to-have 三级排列，生成个性化学习路线
- **STAR 面试题库**：基于岗位要求生成 8-10 道面试题，每题附带 STAR（Situation-Task-Action-Result）答题框架和评估维度
- **ATS 简历优化**：提取岗位高频关键词，建议简历结构、措辞和格式调整，提升机器筛选通过率
- **薪资谈判策略**：根据岗位级别和市场数据，提供谈薪区间和话术建议

## 团队配置

### Agent 1：Job Searcher（岗位搜索专家）

```text
## SOUL.md — Job Searcher

You are an experienced technical recruiter with 10+ years of expertise.

Responsibilities:
- Search job boards via Adzuna API with specific criteria
- Filter low-quality or spam listings
- Score and rank results by relevance
- Extract key skills and requirements from each listing
- Provide market overview (salary trends, experience levels, hot companies)

Model: Claude Sonnet (fast, analytical)
Tools: Adzuna Job Search API

Priority:
1. Roles with detailed, informative job descriptions
2. Positions at reputable companies
3. Listings that clearly state required skills
4. Opportunities with good career growth potential
```

### Agent 2：Skills Advisor（技能顾问）

```text
## SOUL.md — Skills Advisor

You are a career development coach and learning specialist.

Responsibilities:
- Extract ALL required and preferred skills from job listings
- Categorize: technical / tools / soft skills / domain knowledge
- Identify patterns across multiple postings
- Rank skills by frequency and importance (Critical/Important/Nice-to-have)
- Recommend specific courses, books, certifications, and side projects
- Provide realistic timelines (e.g., "Python basics: 2-3 weeks")

Model: Claude Sonnet
Tools: None (analyzes text from Job Searcher)

Output format:
- Priority Skills Matrix with frequency counts
- 30-Day Quick Start Plan (top 3 skills + first steps)
- 3-6 Month Long-term Roadmap with milestones
```

### Agent 3：Interview Coach（面试教练）

```text
## SOUL.md — Interview Coach

You are a senior interview coach and former hiring manager with 1,000+ interviews.

Responsibilities:
- Generate 8-10 interview questions per job listing
- Cover 4 types: Technical / Behavioral / Situational / Role-specific
- Provide STAR framework guidance for each behavioral question
- Include salary negotiation preparation
- Flag common pitfalls to avoid

Model: Claude Opus (strong reasoning)
Tools: None (analyzes text from Job Searcher)

Question format:
  <type>Technical / Behavioral / Situational / Role-specific</type>
  <text>The actual interview question</text>
  <evaluating>What skill or quality they're assessing</evaluating>
  <structure>How to organize your answer</structure>
  <key_points>Critical points to mention</key_points>
  <avoid>Common mistakes or pitfalls</avoid>
```

### Agent 4：Career Advisor（职业策略师）

```text
## SOUL.md — Career Advisor

You are a senior career advisor with 15+ years helping professionals.

Responsibilities:
- Resume: extract ATS keywords, suggest structure, write achievement-focused bullets
- LinkedIn: headline optimization, About section, skills endorsement strategy
- Networking: outreach templates, referral paths, communities to join
- Application: timing, cover letter talking points, follow-up timeline
- Salary: negotiation ranges, counter-offer scripts, total compensation framing

Model: Claude Sonnet
Tools: None (analyzes text from Job Searcher)

Deliverables:
- ATS Keyword List (from all job descriptions)
- Resume Bullet Examples (5-10 tailored bullets)
- LinkedIn Headline Options (3-5 variants)
- 7-Day Action Checklist
```

## 工作流程

```
用户输入 (职位 + 地区 + 数量)
        │
        ▼
┌─────────────────┐
│  Job Searcher    │  ← Adzuna API 搜索
│  搜索并筛选岗位   │
└────────┬────────┘
         │ 岗位列表 + 市场洞察
         ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Skills Advisor   │  │ Interview Coach  │  │ Career Advisor   │
│ 技能差距分析      │  │ 面试题库生成     │  │ 求职策略制定      │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                     │
         ▼                    ▼                     ▼
┌──────────────────────────────────────────────────────────┐
│              合并输出：完整求职报告                         │
│  • 岗位摘要  • 技能路线  • 面试题库  • 投递策略            │
└──────────────────────────────────────────────────────────┘
```

Job Searcher 是唯一使用外部工具的 Agent，其输出通过 CrewAI 的 `context` 参数自动传递给后续三个 Agent。后三个 Agent 可以并行执行，各自从岗位数据中提取不同维度的洞察。

## 设置方法

### 1. 环境准备

```bash
# 克隆项目
git clone https://github.com/byrencheema/job-search-agent.git
cd job-search-agent

# 安装依赖
uv sync

# 配置 API 密钥
cp .env.example .env
```

编辑 `.env`，填入凭证（API 密钥通过环境变量传递，不要硬编码到代码中）：

```bash
ANTHROPIC_API_KEY=sk-ant-...       # Claude API
ADZUNA_APP_ID=12345                # 岗位搜索 API（免费 250 次/月）
ADZUNA_API_KEY=your-adzuna-key
```

### 2. 自定义搜索

编辑 `main.py` 中的参数：

```python
JOB_ROLE = "Machine Learning Engineer"  # 目标职位
LOCATION = "San Francisco"               # 目标地区
NUM_RESULTS = 10                         # 分析岗位数量
```

### 3. 运行

```bash
uv run main.py
```

系统运行 3-5 分钟，自动在 `outputs/` 目录生成：

```
outputs/
├── job_search_report_20260321.txt    # 完整合并报告
├── job_search_20260321.txt           # 岗位列表
├── skills_analysis_20260321.txt      # 技能路线图
├── interview_prep_20260321.txt       # 面试题库
└── career_advisory_20260321.txt      # 求职策略
```

## 进阶：YAML 简历定制系统

如果你需要更深度的简历优化，可以搭配 [claude-code-job-tailor](https://github.com/javiera-vasquez/claude-code-job-tailor)——一个基于 OpenClaw 的简历定制工具。核心思路是**写一次经历，生成无限定制版本**。

### YAML 经历格式

将个人信息、技能和经历写入 YAML 文件，作为所有简历的单一数据源：

```yaml
version: '2.0.0'

personal_info:
  name: 'Zhang Wei'
  titles:
    backend_focused: 'Senior Backend Engineer | Go & Kubernetes'
    ai_focused: 'AI Engineer | NLP & RAG Specialist'
  summaries:
    backend_focused: '8 年分布式系统经验，专注 Go 微服务和 K8s 云原生架构...'
    ai_focused: 'AI 工程师，专注 NLP、语义搜索和 RAG 系统...'

technical_expertise:
  backend:
    - 'Go'
    - 'Python'
    - 'PostgreSQL'
    - 'Redis'
    - 'gRPC'
  ai_machine_learning:
    - 'LangChain'
    - 'Vector Databases'
    - 'RAG'
    - 'Semantic Search'

# 每个岗位方向使用不同的 title 和 summary
# AI 自动根据 JD 选择最相关的经历条目
```

### 3 Agent 定制流程

```
@agent-job-analysis   → 解析 JD，提取加权需求（React: 优先级 10, Python: 优先级 7）
@agent-job-tailor     → 根据加权分数从 YAML 中选择最相关的经历
@agent-tailor-resume  → 生成定制 PDF（modern/classic 两种模板）
```

通过 `/tailor` 命令实时预览，修改立即在浏览器中刷新：

```
/tailor "bytedance"
# → 生成 resume-data/tailor/bytedance/ 目录
#   ├── metadata.yaml      # 岗位元数据
#   ├── job_analysis.yaml  # 加权需求分析
#   ├── resume.yaml        # 定制简历数据
#   └── cover_letter.yaml  # 定制求职信
```

## 自动投递：OpenClaw 求职技能

对于更高度自动化的场景，OpenClaw 社区提供了 `job-auto-apply` 技能，支持多平台自动搜索和投递：

```bash
npx playbooks add skill openclaw/skills --skill job-auto-apply
```

支持平台：LinkedIn（Easy Apply）、Indeed、Glassdoor、ZipRecruiter、Wellfound

关键参数：

```bash
--title "Senior Backend Engineer"    # 目标职位
--location "Remote"                  # 地区或远程
--platforms linkedin,indeed          # 平台选择
--max-applications 10                # 每日投递上限
--min-match-score 0.75               # 匹配度阈值（推荐 0.75+）
--require-confirmation               # 投递前人工确认（强烈推荐）
--dry-run                            # 测试模式，不实际投递
```

安全机制：
- `--require-confirmation` 确保每份申请经过人工审核
- `--dry-run` 模式用于测试流程而不实际提交
- 内置速率限制，遵守平台服务条款
- 不会自动填写你未提供的信息（truthfulness enforcement）

## 国内适配

### 平台差异

国内求职平台（Boss 直聘、拉勾、猎聘、智联招聘）与海外平台有显著区别：

| 维度 | 海外平台 | 国内平台 |
|------|---------|---------|
| 沟通模式 | 投递 → 等回复 | 即时聊天（Boss 直聘的"直聊"模式） |
| 简历格式 | PDF 单页为主 | 平台内建简历模板，PDF 为辅 |
| ATS 系统 | 广泛使用，关键词匹配严格 | 部分大厂使用，中小企业依赖 HR 人工筛选 |
| 投递方式 | 一键 Easy Apply | 需要先打招呼、聊几句再投递 |
| 面试流程 | 电话筛选 → 技术面 → 行为面 | 技术面 → HR 面 → 群面/无领导讨论（校招） |

### 国内面试文化适配

Interview Coach 需要额外覆盖国内特有面试形式：

- **群面（Group Interview）**：5-8 人讨论同一案例，评估团队协作和领导力
- **无领导小组讨论（Leaderless Group Discussion）**：没有指定主持人，考察主动性和结构化思维
- **HR 面**：薪资期望、离职原因、职业规划等软性问题，需要提前准备话术
- **笔试/在线测评**：行测 + 性格测试，部分大厂有专门的编程测试平台

### 中文简历注意事项

- 照片：国内简历通常需要证件照，海外简历不放照片
- 篇幅：国内接受 2 页，应届生建议 1 页
- 个人信息：国内常要求年龄、籍贯、政治面貌（央企/国企），海外简历这些属于隐私
- 项目经历：国内面试非常看重"项目经历"板块，建议用 STAR 法则量化成果

### 国内推送渠道

将求职报告推送到常用工具：

- **飞书/钉钉 Bot**：每日定时推送新匹配岗位和面试准备材料
- **微信（通过 Server 酱）**：关键岗位匹配通知
- **Notion/语雀**：累积求职知识库，记录每次面试复盘

## 关键洞察

- **分工比全能更有效**：4 个专注的 Agent 比 1 个"什么都做"的 Agent 产出质量显著更高——每个 Agent 的提示词可以针对其任务深度优化
- **YAML 单一数据源**：把经历写成结构化 YAML 而不是 Word 文档，AI 就能像查数据库一样按需选择最相关的条目，而不是每次手动改简历
- **加权匹配胜过关键词堆砌**：简历定制的核心不是塞关键词，而是理解岗位描述中哪些要求权重最高（被提及 5 次 vs 1 次），然后优先展示对应经历
- **人工确认不可省略**：自动投递工具必须开启 `--require-confirmation`，错误投递会浪费面试官时间，也会消耗你在平台上的信用
- **面试准备要按岗位定制**：通用面试题的价值有限，基于实际 JD 生成的题目能精准覆盖面试官最可能考察的方向
- **从分析开始，不是从投递开始**：先用 Agent 分析 10 个目标岗位的共性要求，校准自己的技能差距，再定制简历和投递——这比盲目海投的转化率高一个数量级

## 灵感来源

这个用例综合了多个社区实践。[Byron Cheema](https://github.com/byrencheema/job-search-agent) 在 UC Irvine Claude Builder Club 的工作坊上搭建了 CrewAI + Claude 的 4 Agent 求职系统（Job Searcher / Skills Advisor / Interview Coach / Career Advisor），通过 Adzuna API 拉取实时岗位，生成完整的求职分析报告。[Javiera Vasquez](https://github.com/javiera-vasquez/claude-code-job-tailor) 开发的 claude-code-job-tailor 工具将简历数据存储为 YAML 格式，用 3 个专业 Agent 实现"60 秒内从 JD 到定制 PDF"的全流程。OpenClaw 社区的 [`job-auto-apply`](https://playbooks.com/skills/openclaw/skills/job-auto-apply) 技能进一步支持 LinkedIn、Indeed 等平台的自动搜索和投递。此外，多位用户在实际求职中验证了这些方法的效果：有人报告每天获得 4-8 个高匹配岗位推荐，也有人在 12 小时内完成了 1000+ 份申请的定制投递。

## 相关链接

- [byrencheema/job-search-agent](https://github.com/byrencheema/job-search-agent) — CrewAI + Claude 4 Agent 求职系统
- [javiera-vasquez/claude-code-job-tailor](https://github.com/javiera-vasquez/claude-code-job-tailor) — YAML 简历定制 + 实时预览
- [OpenClaw job-auto-apply 技能](https://playbooks.com/skills/openclaw/skills/job-auto-apply) — 多平台自动投递
- [OpenClaw resume-cv-builder 技能](https://playbooks.com/skills/openclaw/skills/resume-cv-builder) — 简历生成器
- [Adzuna Job Search API](https://developer.adzuna.com/) — 免费岗位搜索 API（250 次/月）
- [CrewAI 文档](https://docs.crewai.com/) — 多智能体编排框架

---

**原文链接**：本用例为中文原创，综合多个英文社区项目和实践经验撰写。
