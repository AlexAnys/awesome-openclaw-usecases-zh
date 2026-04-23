# 自主科研流水线：从灵感到论文

> 含国内适配：国产 GPU 支持现状、学术数据库差异、NanoResearch 中文替代方案

你有一个研究想法——比如"图神经网络在药物发现中的应用"——但从文献调研到实验设计、代码编写、GPU 训练、结果分析、论文撰写再到同行评审，整个流程可能耗费数周。你希望把这些重复性工作交给 AI，自己专注于核心创新。

[AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw)（9K+ stars，MIT 许可）是一个 23 阶段的全自主科研流水线。你输入一句话的研究主题，它输出一篇会议级别的 LaTeX 论文——包含真实文献引用、可运行的实验代码、自动生成的图表和多智能体同行评审。

## 与 arXiv 论文阅读用例的区别

本仓库已有 arXiv 论文阅读用例（Prismer-AI），两者互补但定位完全不同：

| 维度 | arXiv 论文阅读 | 自主科研流水线（本用例） |
|------|--------------|----------------------|
| 方向 | **读**论文 — 摘要、解析、归类 | **写**论文 — 从零产出完整论文 |
| 输入 | arXiv 论文 URL 或关键词 | 一句话研究主题 |
| 输出 | 结构化笔记、文献综述 | 完整 LaTeX 论文 + 实验代码 + 图表 |
| 复杂度 | 单一信息提取任务 | 23 阶段多智能体流水线 |
| 资源 | 仅需 LLM API | LLM API + GPU/CPU 计算资源 |

建议先用论文阅读用例做文献调研，再用本用例启动完整科研流程。

## 23 阶段流水线概览

```
阶段 A: 研究定义                     阶段 E: 实验执行
  1. 主题初始化                        12. 实验运行（沙箱环境）
  2. 问题分解                          13. 迭代修复 ← 自愈机制

阶段 B: 文献发现                     阶段 F: 分析与决策
  3. 搜索策略制定                      14. 结果分析 ← 多智能体辩论
  4. 文献收集（OpenAlex/S2/arXiv）     15. 研究决策 ← PROCEED/REFINE/PIVOT
  5. 文献筛选 [门控]
  6. 知识提取                         阶段 G: 论文撰写
                                       16. 论文大纲
阶段 C: 知识综合                       17. 论文初稿（5000-6500 词）
  7. 综合分析                          18. 同行评审 ← 方法-证据一致性检查
  8. 假说生成 ← 多视角辩论             19. 论文修订

阶段 D: 实验设计                     阶段 H: 定稿发布
  9. 实验方案设计 [门控]               20. 质量门控 [门控]
 10. 代码生成（硬件感知）              21. 知识归档
 11. 资源规划                         22. LaTeX 导出（NeurIPS/ICML/ICLR 模板）
                                       23. 引用验证（arXiv/CrossRef/DataCite）
```

**门控阶段**（5、9、20）默认暂停等待人工确认，使用 `--auto-approve` 可跳过。

**决策循环**：阶段 15 可触发 REFINE（回到阶段 13 调参）或 PIVOT（回到阶段 8 换方向），产物自动版本化。

## 核心能力

| 能力 | 说明 |
|------|------|
| 真实文献 | 从 OpenAlex、Semantic Scholar、arXiv 检索真实论文，非 LLM 编造 |
| 硬件感知 | 自动检测 NVIDIA CUDA / Apple MPS / CPU，据此调整生成的实验代码 |
| 自愈实验 | 实验失败时自动分析错误、修复代码并重跑 |
| 多智能体辩论 | 假说生成、结果分析、同行评审均采用多视角结构化辩论 |
| 引用验证 | 4 层验证（arXiv、CrossRef、DataCite、LLM），99.7% 引用真实率 |
| AI-slop 检测 | 质量审计阶段检测 AI 特征性废话，确保论文质量 |

## 产出物

每次运行生成一个完整的工作空间：

| 文件 | 说明 |
|------|------|
| `paper_draft.md` | 完整论文（引言、相关工作、方法、实验、结论） |
| `paper.tex` | 会议级 LaTeX（NeurIPS/ICML/ICLR 模板） |
| `references.bib` | 真实 BibTeX 引用，自动剪枝至正文引用 |
| `experiment runs/` | 实验代码 + 沙箱运行结果 + JSON 格式指标 |
| `charts/` | 自动生成的对比图、误差条、置信区间 |
| `reviews.md` | 多智能体同行评审报告 |
| `deliverables/` | 打包产出，可直接导入 Overleaf |

Showcase 数据：8 篇论文横跨 8 个领域（数学、统计、生物、计算、NLP、RL、视觉、鲁棒性），共 121 页，291 条引用（99.7% 验证通过），54,348 行代码，总运行时间约 27 小时。

## MetaClaw 跨轮学习

v0.3.0 引入 [MetaClaw](https://github.com/aiming-lab/MetaClaw) 集成——流水线失败的经验会转化为结构化技能，注入后续运行的所有 23 个阶段：

```
运行 1 失败 → 提取教训 → 生成 arc-* 技能文件
                                    ↓
运行 2 启动 → 注入技能 → 避免重蹈覆辙
```

实测效果：重试率降低 24.8%，修复循环减少 40%，整体鲁棒性提升 18.3%。

配置方式：

```yaml
metaclaw_bridge:
  enabled: true
  lesson_to_skill:
    enabled: true
    min_severity: "warning"
    max_skills_per_run: 5
```

## 所需技能与资源

- OpenClaw + `web_search`（内置）
- LLM API（OpenAI GPT-4o 或兼容接口）
- 计算资源（见下表）

### GPU 资源需求

| 运行模式 | 硬件要求 | 适用场景 |
|---------|---------|---------|
| `simulated` | 仅 CPU，无 GPU | 论文写作为主，实验用模拟数据 |
| `sandbox` | 单 GPU（8GB+ 显存） | 小规模真实实验，推荐入门 |
| `docker` | Docker 环境 + GPU | 隔离环境运行实验 |
| `ssh_remote` | 远程 GPU 服务器 | 大规模训练，通过 SSH 提交 |
| `colab_drive` | Google Colab + Drive | 免费 GPU，适合无本地 GPU 用户 |

Apple Silicon 用户：MPS 后端自动检测，可在 sandbox 模式下运行中小规模实验。

## 如何设置

### OpenClaw 启动（推荐）

> 前提条件：需要已配置 LLM API Key（如 `OPENAI_API_KEY`），`simulated` 以外的模式还需要对应的计算资源。

1. 将仓库 URL 分享给 OpenClaw：
   ```
   https://github.com/aiming-lab/AutoResearchClaw
   ```
2. OpenClaw 自动读取 `RESEARCHCLAW_AGENTS.md`，理解整个流水线。
3. 发送研究主题：
   ```text
   Research the application of graph neural networks in drug discovery
   ```
4. OpenClaw 完成：克隆 → 安装 → 配置 → 运行 23 阶段 → 返回论文。
   首次运行需要安装依赖，后续运行会复用已有环境。

### 手动 CLI 启动

```bash
git clone https://github.com/aiming-lab/AutoResearchClaw.git
cd AutoResearchClaw
python3 -m venv .venv && source .venv/bin/activate
pip install -e .

researchclaw setup        # 交互式安装检查
researchclaw init          # 创建 config.arc.yaml
export OPENAI_API_KEY="sk-..."
researchclaw run --topic "Your research idea" --auto-approve
```

产出目录：`artifacts/rc-YYYYMMDD-HHMMSS-<hash>/deliverables/`

### ACP 模式（使用 Claude Code / Codex / Gemini CLI）

AutoResearchClaw 支持任何 ACP 兼容的编程 agent 作为 LLM 后端，无需单独配置 API key：

```yaml
llm:
  provider: "acp"
  acp:
    agent: "claude"   # 或 codex / gh / gemini / opencode / kimi
```

### OpenClaw Bridge 高级配置

```yaml
openclaw_bridge:
  use_cron: true              # 定时科研任务
  use_message: true           # 进度通知（Discord/Slack/Telegram）
  use_memory: true            # 跨会话知识持久化
  use_sessions_spawn: true    # 并行子会话
  use_web_fetch: true         # 文献检索时实时搜索
```

## 中国用户适配

### 国产 GPU 支持现状

| GPU 平台 | 支持状态 | 说明 |
|---------|---------|------|
| NVIDIA CUDA | 完全支持 | AutoResearchClaw 原生支持 |
| Apple MPS | 完全支持 | 自动检测，适合 Mac 用户 |
| **华为昇腾 (Ascend)** | 需适配 | PyTorch 昇腾版（torch_npu）可运行，但需手动配置实验代码模板 |
| **寒武纪 MLU** | 需适配 | 需 Cambricon PyTorch 适配层，社区暂无现成方案 |
| **海光 DCU** | 需适配 | ROCm 兼容层理论可用，实测案例较少 |

**建议**：国产 GPU 用户优先使用 `simulated` 模式完成论文写作，实验部分在本地昇腾/寒武纪环境手动运行。

### 学术数据库差异

AutoResearchClaw 的文献检索依赖 OpenAlex、Semantic Scholar 和 arXiv，均为国际开放数据库。国内学术资源的覆盖情况：

| 数据源 | 中文论文覆盖 | 访问限制 |
|--------|------------|---------|
| OpenAlex | 收录部分中文期刊 | 无限制 |
| Semantic Scholar | 有限 | 无限制 |
| arXiv | 仅英文 | 无限制 |
| **知网 (CNKI)** | 最全 | 付费，无公开 API |
| **万方数据** | 较全 | 付费，API 受限 |
| **百度学术** | 聚合搜索 | 免费，反爬严格 |

**现状**：如果你的研究主题以中文文献为主（如中医药、中国经济等），AutoResearchClaw 的文献覆盖会不足。可考虑手动补充知网文献，或使用下方的 NanoResearch 替代方案。

### NanoResearch：中文优先的替代方案

[NanoResearch](https://github.com/OpenRaiser/NanoResearch)（280+ stars，MIT 许可）是另一个端到端自主科研系统，中文优先设计，核心差异在于**真正执行 GPU 训练**：

| 维度 | AutoResearchClaw | NanoResearch |
|------|-----------------|-------------|
| 语言 | 英文优先，有中文文档 | **中文优先**，有英文文档 |
| 流水线 | 23 阶段，更精细 | 9 阶段，更简洁 |
| GPU 执行 | 沙箱/模拟模式 | **原生 SLURM 集群提交** |
| 实验证据 | 沙箱结果 | 真实 GPU 训练日志 |
| 生态 | OpenClaw 深度集成 | 独立运行为主 |
| 成熟度 | 9K+ stars，社区活跃 | 280+ stars，快速迭代 |

**适用场景**：如果你有 SLURM 集群（高校/实验室常见配置），且研究方向偏实验密集型（深度学习训练），NanoResearch 的真实 GPU 执行能力更有优势。

### LLM API 替代

```yaml
# 使用国内 LLM API 替代 OpenAI
llm:
  base_url: "https://dashscope.aliyuncs.com/compatible-mode/v1"  # 通义千问
  api_key_env: "DASHSCOPE_API_KEY"
  primary_model: "qwen-max"
  fallback_models: ["qwen-plus"]
```

其他可选：智谱 GLM-4、月之暗面 Kimi、DeepSeek 等 OpenAI 兼容接口。

### 推送渠道适配

| 原版方案 | 国内替代 | 说明 |
|---------|---------|------|
| Discord | **飞书机器人** | 支持富文本卡片，适合实验室 |
| Discord | **钉钉群机器人** | Webhook 方式，最简单 |
| Slack | **企业微信** | 企业/高校用户首选 |

## 实用建议

- **从 `simulated` 模式开始**：首次使用先跑 simulated 模式，验证文献检索和论文撰写流程跑通后再升级到 `sandbox`/`docker` 做真实实验
- **门控阶段不要跳过**：阶段 5（文献筛选）、9（实验方案）、20（质量门控）默认暂停等人工确认，这些是最值得人工把关的环节。`--auto-approve` 适合熟悉流程后使用
- **API 成本预期**：单次完整运行（23 阶段）消耗约 $5-15 的 LLM API token（视模型和主题复杂度），文献检索阶段消耗最多
- **论文产出需人工打磨**：AI 生成的初稿框架和实验基线质量不错，但核心创新论述、实验分析深度、写作风格仍需研究者自行修改
- **引用验证值得信赖**：4 层验证机制（arXiv/CrossRef/DataCite/LLM）实测 99.7% 引用真实率，但建议投稿前仍做一轮人工核验

## 学术诚信提醒

使用 AI 辅助科研工具时，务必注意以下合规要求：

1. **必须声明 AI 使用**：2026 年起，主要学术出版商（Elsevier、Springer Nature、Wiley 等）均要求在论文中明确声明 AI 工具的使用范围——包括数据分析、文本生成、图表制作等环节。
2. **AI 不能列为作者**：所有主要期刊和会议（NeurIPS、ICML、ICLR 等）明确禁止将 AI 列为论文作者。
3. **人类负全部责任**：论文的准确性、原创性和学术诚信由人类作者全权负责。AI 生成的内容必须经过人工审核。
4. **完全由 AI 撰写的论文会被拒稿**：AutoResearchClaw 的定位是**辅助工具**，产出的论文需要人类研究者审阅、修改和把关。
5. **实验可复现性**：确保 AI 生成的实验代码和结果可独立复现，这是学术论文的基本要求。

**建议做法**：将 AutoResearchClaw 视为"超级科研助手"而非"论文代写工具"——用它加速文献调研、生成实验基线、产出初稿框架，但核心创新和最终论文质量由你把关。

## 相关链接

- [AutoResearchClaw GitHub](https://github.com/aiming-lab/AutoResearchClaw) — 23 阶段自主科研流水线
- [MetaClaw](https://github.com/aiming-lab/MetaClaw) — 跨轮学习框架
- [NanoResearch](https://github.com/OpenRaiser/NanoResearch) — 中文优先，SLURM 集成的替代方案
- [AutoResearchClaw Showcase](https://github.com/aiming-lab/AutoResearchClaw/blob/main/docs/showcase/SHOWCASE.md) — 8 篇自主生成论文展示
- [集成指南](https://github.com/aiming-lab/AutoResearchClaw/blob/main/docs/integration-guide.md) — OpenClaw 集成详细文档

---

**项目链接**：[AutoResearchClaw on GitHub](https://github.com/aiming-lab/AutoResearchClaw)
