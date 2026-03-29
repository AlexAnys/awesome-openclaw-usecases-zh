# 竞争对手分析与价格监控

> 含国内适配：百度指数 / 微信指数 / 飞书推送 / SerpAPI 替代方案

用 Perplexity MCP（联网搜索）和 Firecrawl MCP（网页抓取）构建一个持续运行的竞品监控智能体。它每周自动扫描竞品动态，也支持按专题随时深挖，把原来每月 $150+ 的竞品分析订阅和 10 小时人工调研，压缩到每月约 $1.20 和 6 分钟。

## 功能介绍

- **周报模式**：每周自动扫描预设的竞品列表，输出定价变动、功能更新、内容策略等关键动态
- **专题模式**：针对特定话题（如"竞品 AI 功能上线"）进行即时深度分析
- **差距识别**：不只是罗列数据——自动对比竞品覆盖了什么、遗漏了什么，找出你的机会窗口
- **结构化输出**：返回 JSON 格式的竞品分析报告，包含来源引用，可直接对接后续工作流
- **成本极低**：Perplexity 免费层每天 5 次搜索，Firecrawl 每月 500 次免费抓取，日常监控基本零成本

## 痛点

你订阅了 Semrush、SimilarWeb 或 Crayon 等竞品分析工具，每月花费 $150 以上。但大部分功能你用不到，真正需要的只是"竞品这周做了什么"。你还得花几个小时手动浏览竞品的博客、定价页和更新日志，把发现整理成报告。这些工作重复、琐碎，而且一旦忙起来就容易断掉——竞品推出了重要功能，你可能一个月后才发现。

## 所需技能

- [Perplexity MCP](https://github.com/ppl-ai/modelcontextprotocol) — 联网搜索，支持多源聚合和引用（需 API Key）
- [Firecrawl MCP](https://github.com/mendableai/firecrawl-mcp-server) — 网页抓取与结构化提取（需 API Key）

## 如何设置

### 1. 获取 API 密钥

- Perplexity API Key：在 [perplexity.ai/settings/api](https://perplexity.ai/settings/api) 获取（免费层每天 5 次搜索）
- Firecrawl API Key：在 [firecrawl.dev](https://www.firecrawl.dev/) 注册获取（免费层每月 500 次抓取）

### 2. 配置 MCP 服务器

```json
{
  "mcpServers": {
    "perplexity": {
      "command": "npx",
      "args": ["-y", "@perplexity-ai/mcp-server"],
      "env": {
        "PERPLEXITY_API_KEY": "${PERPLEXITY_API_KEY}"
      }
    },
    "firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "${FIRECRAWL_API_KEY}"
      }
    }
  }
}
```

> 将 API 密钥存放在环境变量中，不要硬编码在配置文件里。

### 3. 创建竞品监控列表

在项目中创建 `.claude/research-profiles/competitor-watchlist.md`，定义你要监控的竞品：

```markdown
# 竞品监控列表

## 直接竞品（Direct Competitors）
- CompanyA — https://companya.com — 主要产品：XXX
- CompanyB — https://companyb.com — 主要产品：YYY

## 间接竞品（Indirect Competitors）
- CompanyC — https://companyc.com — 切入角度：ZZZ

## 工具类竞品（Tool Competitors）
- ToolX — https://toolx.dev — 开发者工具

## 监控重点
- 定价页变动（/pricing）
- 产品更新日志（/changelog, /blog）
- 新功能公告
- 招聘动态（可反映战略方向）
```

### 4. 添加智能体指令

创建 `/competitive-check` 命令，将以下内容添加到 OpenClaw 设置中：

```text
You are a competitive intelligence analyst. When I say /competitive-check,
do the following:

1. Load the competitor watchlist from .claude/research-profiles/competitor-watchlist.md
2. For each competitor:
   a. Use Perplexity to search for recent news, product updates, and pricing changes
      in the past 7 days
   b. Use Firecrawl to scrape their pricing page and changelog for exact details
3. Compare findings against our product positioning
4. Output a structured report in JSON with these fields:
   - competitor_name
   - changes_detected (array of {type, summary, url, date})
   - pricing_changes (if any)
   - content_gaps (topics they covered that we haven't)
   - recommended_actions (array of suggested responses)
   - sources (array of URLs)

When I say /competitive-check --topic "xxx", focus the analysis on that
specific topic across all competitors instead of a general weekly scan.

Rules:
- Always include source URLs for every claim
- Flag any pricing decrease > 10% as HIGH PRIORITY
- If a competitor launches a feature we don't have, mark it as OPPORTUNITY
- Keep the report actionable — every finding should have a suggested response
```

### 5. 设置定时执行（可选）

如果需要每周自动执行，可以通过 OpenClaw 内置的定时任务功能触发：

```bash
openclaw cron add \
  --name "competitive-weekly" \
  --cron "0 9 * * 1" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "/competitive-check"
```

或者直接在对话中用自然语言设置：

```text
帮我创建一个定时任务：每周一早上 9 点执行 /competitive-check，生成竞品周报。
时区设为 Asia/Shanghai，使用独立会话。
```

## 真实案例

**周报模式：**

```
你：/competitive-check
```

OpenClaw 加载竞品列表，依次搜索和抓取，约 45 秒后返回：

> **竞品周报 — 2026-03-24**
>
> **CompanyA**
> - [定价变动] 专业版从 $49/月降至 $39/月（-20%） ⚠️ HIGH PRIORITY
> - [新功能] 上线 AI 自动摘要功能
> - 建议：评估是否跟进降价；AI 摘要功能我们已在开发中，考虑提前发布
>
> **CompanyB**
> - [内容策略] 发布了 3 篇关于"企业级安全"的博客
> - [招聘] 在招 3 名后端工程师（Go），疑似在做基础设施重构
> - 建议：我们尚未覆盖企业安全话题，建议产出一篇对标内容
>
> **内容差距：** 本周竞品覆盖了"SOC 2 合规"和"数据驻留"话题，我们均未涉及。
>
> 来源：[12 条引用链接]

**专题模式：**

```
你：/competitive-check --topic "AI 定价策略"
```

> 针对 AI 定价策略的竞品分析：
>
> - CompanyA：按 token 计费，GPT-4 调用 $0.03/次
> - CompanyB：固定月费 + 用量上限，超出后按量计费
> - CompanyC：完全免费增值模式，靠企业版盈利
>
> **我们的机会：** 目前没有竞品提供"按结果付费"模式（只为成功的 AI 输出计费）。这可能是一个差异化定价角度。

## 实用建议

- **先窄后宽**：刚开始只监控 3-5 个直接竞品，跑顺之后再扩展到间接竞品和工具类竞品。
- **定价页是金矿**：竞品定价变动往往比功能更新更能反映战略方向。用 Firecrawl 定期抓取定价页，对比历史快照。
- **招聘信息是情报**：竞品在招什么人，往往暗示未来 3-6 个月的产品方向。
- **输出对接后续动作**：将 JSON 报告接入飞书/Slack webhook，让团队第一时间看到关键变动。
- **控制成本**：Perplexity 免费层每天 5 次搜索，监控 5 个竞品刚好够用。Firecrawl 免费层每月 500 次抓取，按周监控 10 个页面绰绰有余。

## 中国用户适配

### 搜索能力替代

Perplexity 在国内访问可能不稳定。以下替代方案可以实现类似的联网搜索能力：

| 方案 | 适用场景 | 说明 |
|------|---------|------|
| [SerpAPI MCP](https://github.com/serpapi/serpapi-mcp-server) | 通用搜索 | 支持百度、Google 等多搜索引擎，有免费额度 |
| [Tavily MCP](https://github.com/tavily-ai/tavily-mcp) | AI 优化搜索 | 为 AI Agent 设计的搜索 API，结果质量高 |
| 百度搜索 MCP | 国内搜索 | 社区有多个实现，搜索"baidu mcp server" |

### 国内数据源补充

在标准竞品监控之外，建议补充以下国内特有数据源：

- **百度指数**（index.baidu.com）：监控竞品品牌词搜索趋势，趋势突变往往意味着重大动作
- **微信指数**：微信小程序内查看，反映竞品在微信生态的声量变化
- **巨量算数**（trendinsight.oceanengine.com）：抖音/头条生态数据，适合 C 端产品竞品监控
- **天眼查 / 企查查**：监控竞品融资、人员变动、专利申请等企业动态
- **36氪 / IT桔子**：行业新闻和融资数据

### 推送渠道适配

将竞品周报推送到国内常用的团队协作工具：

```text
After completing the competitive analysis, send the report summary
to our team channel:
- Format the key findings as a structured message
- Include HIGH PRIORITY items at the top
- Add source links for verification
- Use Feishu/Lark webhook: ${FEISHU_WEBHOOK_URL}
```

飞书 Webhook 推送示例：

```bash
# 通过飞书自定义机器人推送竞品周报
curl -X POST "${FEISHU_WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "interactive",
    "card": {
      "header": {
        "title": {"tag": "plain_text", "content": "竞品周报 - 2026-03-24"},
        "template": "orange"
      },
      "elements": [
        {
          "tag": "markdown",
          "content": "**⚠️ CompanyA 专业版降价 20%**\n详情见完整报告"
        }
      ]
    }
  }'
```

> 钉钉用户替换为钉钉 Webhook URL，消息格式类似。企业微信同理。

### 国内竞品监控工具现状

目前国内的竞品监控工具（如鹰眼监测、新榜等）主要聚焦舆情和内容监控，在产品功能对比和定价追踪方面能力有限。用 OpenClaw + MCP 的方案优势在于：

- **高度可定制**：你决定监控什么、怎么分析、输出什么格式
- **成本可控**：免费层即可覆盖小团队需求，不用订阅昂贵的 SaaS
- **可扩展**：轻松接入新数据源，不受平台功能限制

## 与"市场调研与产品工厂"的区别

| 维度 | 市场调研与产品工厂 | 竞争对手分析与价格监控 |
|------|-------------------|----------------------|
| 目标 | 从零发现产品机会并构建 MVP | 对已知竞品进行持续跟踪 |
| 频率 | 一次性或偶尔执行 | 每周自动执行，持续运行 |
| 数据源 | Reddit、X 上的用户痛点 | 竞品官网、定价页、更新日志 |
| 输出 | 产品创意 + MVP 代码 | 结构化竞品情报报告 |
| 适用阶段 | 产品从 0 到 1 | 产品上线后的持续竞争 |

## 相关链接

- [Perplexity MCP Server](https://github.com/ppl-ai/modelcontextprotocol)
- [Firecrawl MCP Server](https://github.com/mendableai/firecrawl-mcp-server)
- [SerpAPI Competitive Intelligence Agent](https://github.com/serpapi/competitive-intelligence-agent) — SerpAPI 官方竞品分析 Agent 参考实现
- [Competitive Repricing Agent MCP](https://github.com/ajeetraina/competitive-repricing-agent-mcp) — 价格监控与自动报告 MCP 示例

---

**原文链接**：[The Claude AI Agent That Replaced My $150/Month Competitor Analysis Tool](https://genaiunplugged.substack.com/p/competitive-ai-analysis-agent)
