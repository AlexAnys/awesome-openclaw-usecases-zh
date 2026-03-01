# A 股智能分析助手

每天开盘前看一堆研报、盘中盯着行情软件、收盘后复盘总结——散户和个人投资者在信息收集和分析上花的时间，远比做决策的时间多。

这个用例让 OpenClaw 成为你的 A 股分析搭档：自动拉取实时行情和技术指标，用 AI 做多维度分析，把结论推送到你的手机上。你专注决策，它负责跑数据。

## 它能做什么

- **实时行情监控**：获取沪深 A 股、ETF、可转债的实时报价和历史数据
- **技术分析**：自动计算 MA、MACD、RSI、KDJ、布林带等指标，生成 K 线图
- **AI 研判**：结合技术指标和基本面数据，给出买入/持有/观望建议及置信度
- **资金流向追踪**：监控主力资金、北向资金动态
- **财报快读**：拉取个股财务数据（PE/PB、ROE、利润率、分红记录），快速定位异常
- **定时推送**：盘前分析和盘后复盘自动发送到飞书/钉钉/企业微信

## 所需技能

核心数据层（选一个即可）：

- [akshare-stock-skill](https://github.com/DAnonyvinciT/akshare-stock-skill) —— 多维评分 + 技术指标 + SQLite 缓存，适合日常使用
- [stock-daily-analysis-skill](https://github.com/chjm-ai/stock-daily-analysis-skill) —— LLM 驱动的每日分析，支持 DeepSeek/Gemini，适合想要 AI 研判的用户
- [openclaw-stock-skill](https://github.com/molezzz/openclaw-stock-skill) —— 自然语言查询路由，支持 18 种查询类型

所有技能底层都依赖 [AkShare](https://github.com/akfamily/akshare)（16,000+ stars），一个成熟的中国金融数据 Python 库。

可选增强：

- [stock-mcp](https://github.com/huweihua123/stock-mcp)（92 stars）—— MCP 协议金融分析服务器，支持 A 股/港股/美股，有健康度评分系统
- IM 推送插件：飞书/钉钉/企业微信（参考对应的 IM 集成用例）

## 如何设置

### 方案一：快速上手（AkShare 技能 + 对话式分析）

1. 安装 AkShare 数据技能：

```bash
# 安装 Python 依赖
pip install akshare matplotlib

# 克隆技能到本地
git clone https://github.com/DAnonyvinciT/akshare-stock-skill.git
cp -r akshare-stock-skill ~/.openclaw/workspace/skills/
# 注意：技能目录路径取决于你的 OpenClaw workspace 配置，
# 默认为 ~/.openclaw/workspace/skills/，首次使用时在对话中
# 说"查看已安装技能"确认加载成功
```

2. 直接用自然语言查询：

```text
帮我看一下贵州茅台（600519）最近的走势，包括技术指标和资金流向。

今天 A 股大盘表现怎么样？半导体板块有什么异动？

帮我分析一下宁德时代的最新财报，重点看 ROE 和利润率变化。
```

### 方案二：每日自动分析 + 推送

1. 安装 LLM 分析技能：

```bash
git clone https://github.com/chjm-ai/stock-daily-analysis-skill.git
cp -r stock-daily-analysis-skill ~/.openclaw/workspace/skills/
```

2. 配置 AI 模型（推荐 DeepSeek，国内访问稳定）：

```text
我想设置一个每日股票分析任务。

每个交易日早上 8:30，分析以下股票的技术面和基本面：
- 600519（贵州茅台）
- 300750（宁德时代）
- 000858（五粮液）

分析完成后通过飞书发给我，包括：
1. 每只股票的技术指标摘要（MA、MACD、RSI）
2. 资金流向（主力净流入/流出）
3. AI 给出的操作建议和置信度
4. 今日需要关注的风险点
```

3. 设置定时任务：

```bash
openclaw cron add \
  --name "daily-a-share-analysis" \
  --cron "30 8 * * 1-5" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "执行今日股票分析并推送" \
  --announce \
  --channel feishu
```

### 方案三：MCP 协议深度分析

适合需要多市场、多数据源融合分析的进阶用户：

```bash
git clone https://github.com/huweihua123/stock-mcp.git
cd stock-mcp
pip install -r requirements.txt
```

在 `openclaw.json` 中添加 MCP 服务器配置，例如：

```json
{
  "mcpServers": {
    "stock-mcp": {
      "command": "python",
      "args": ["-m", "stock_mcp.server"],
      "cwd": "/path/to/stock-mcp"
    }
  }
}
```

配置完成后即可使用深度研究功能：

```text
对比分析宁德时代和比亚迪最近一个季度的财务健康度，
包括估值水平、盈利能力、现金流状况，给出综合评分。
```

## 实用建议

- **数据源选择**：AkShare 免费且覆盖全面，日常分析足够。如需更高频数据（分钟级），可申请 Tushare Pro 积分（API key 通过环境变量 `TUSHARE_TOKEN` 传递，不要硬编码在配置文件中）
- **AI 模型推荐**：DeepSeek 对中文金融文本理解好且国内访问稳定，是首选；也可用 Gemini 或 Claude
- **云端部署**：如果需要 24 小时盯盘，推荐部署到阿里云/腾讯云轻量服务器（月费几十元），配合 cron 实现全自动
- **K 线图导出**：akshare-stock-skill 和 openclaw-stock-skill 都支持生成 K 线图片，可直接在 IM 消息中查看
- **风险提醒**：AI 分析仅供参考，不构成投资建议。建议设置止损线和仓位上限，不要完全依赖自动化决策

> **⚠️ 免责声明**：本用例仅用于个人投资研究辅助，AI 给出的分析和建议不构成投资建议。股市有风险，投资需谨慎。

## 相关链接

- [akshare-stock-skill - GitHub](https://github.com/DAnonyvinciT/akshare-stock-skill)（多维评分 + 缓存）
- [stock-daily-analysis-skill - GitHub](https://github.com/chjm-ai/stock-daily-analysis-skill)（LLM 每日分析）
- [openclaw-stock-skill - GitHub](https://github.com/molezzz/openclaw-stock-skill)（自然语言查询）
- [stock-mcp - GitHub](https://github.com/huweihua123/stock-mcp)（MCP 多市场分析）
- [AkShare 官方文档](https://akshare.akfamily.xyz/)
- [阿里云 - 零门槛部署 OpenClaw 接入 A 股数据](https://developer.aliyun.com/article/1713167)
- [腾讯云 - 一键部署 OpenClaw 股市分析师](https://blog.csdn.net/u012263509/article/details/157762036)
- [新浪财经 - 百度智能云跑 OpenClaw，AI 帮我 24 小时盯盘](https://finance.sina.com.cn/tech/roll/2026-02-12/doc-inhmpusi6882680.shtml)
