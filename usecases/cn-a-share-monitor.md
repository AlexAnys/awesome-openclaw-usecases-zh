# A 股每日行情监控

> 与 [AI 财报追踪器](earnings-tracker.md) 互补：财报追踪器关注单个公司的季报事件，本用例关注每个交易日的市场全局动态。

每天开盘前翻研报、盘中盯行情软件、收盘后复盘——个人投资者在信息收集上花的时间远超做决策的时间。如果 AI 能每天帮你整理大盘走势、板块异动、资金流向，你只需要看结论就行。

这个用例让 OpenClaw 每个交易日自动采集 A 股行情数据，整理成结构化简报推送给你。

## 它能做什么

- **盘前简报**：隔夜外盘走势、今日重要事件（如 LPR 调整、IPO 申购）提醒
- **盘后复盘**：大盘指数涨跌、成交量对比、涨跌家数统计
- **板块追踪**：按行业板块排序当日涨跌幅，识别热点板块轮动
- **资金流向**：主力资金净流入/流出 TOP 排名、北向资金动态
- **自选股监控**：你关注的股票当日表现、技术指标变化

## 能力边界（基于社区实测）

根据多位开发者的真实使用反馈，**信息采集和数据整理是 OpenClaw 在 A 股场景中最可靠的能力**。以下是实测验证的能力边界：

| 场景 | 可靠性 | 说明 |
|------|--------|------|
| 行情数据采集 | ✅ 可靠 | AKShare 接口稳定，数据覆盖全面 |
| 数据整理推送 | ✅ 可靠 | 结构化输出 + 定时 cron 推送 |
| 技术指标计算 | ✅ 可靠 | MA/MACD/RSI 等基于公式计算，结果确定 |
| 舆情/新闻聚合 | ⚠️ 基本可用 | 依赖浏览器自动化，偶尔有反爬限制 |
| AI 买卖建议 | ❌ 不推荐依赖 | LLM 调用延迟 2-5 秒，Chrome 连接不稳定，决策黑箱 |

> 引用来源：[Jason Zhang - 用 OpenClaw 做 A 股量化？我试了试，聊聊真实感受](https://junxinzhang.com/openclaw-quant-trading-a-share/)（12 年 IT 经验，3 年美股量化经验的开发者实测）

## 所需技能

**数据层**（核心依赖）：

[AKShare](https://github.com/akfamily/akshare)（16,000+ stars，MIT 开源）—— 中国金融数据的事实标准库，覆盖沪深 A 股、港股、ETF、可转债、宏观经济指标。免费，无需 API Key。

可选的 MCP 方案（免去写 Python 脚本）：

- [mcp-cn-a-stock](https://github.com/elsejj/mcp-cn-a-stock)（400+ stars）—— 专门为 LLM 设计的 A 股数据 MCP 服务，OpenClaw 可直接调用

**推送层**：

飞书 / 钉钉 / 企业微信（参考对应的 IM 集成用例）

## 如何设置

### 方案一：MCP 服务 + 自然语言查询（推荐）

无需写代码，通过 MCP 服务让 OpenClaw 直接获取 A 股数据：

1. 安装 MCP 服务：

```bash
git clone https://github.com/elsejj/mcp-cn-a-stock.git
cd mcp-cn-a-stock
pip install -r requirements.txt
```

2. 在 OpenClaw 配置中添加 MCP 服务器：

```json
{
  "mcpServers": {
    "cn-a-stock": {
      "command": "python",
      "args": ["-m", "mcp_cn_a_stock"],
      "cwd": "/path/to/mcp-cn-a-stock"
    }
  }
}
```

3. 直接用自然语言查询：

```text
今天 A 股大盘表现怎么样？上证和深证的涨跌情况。

半导体板块最近一周的走势如何？

帮我看一下北向资金今天的流入流出情况。
```

### 方案二：AKShare + Python 沙箱

适合想要更灵活控制数据处理逻辑的用户：

```bash
pip install akshare
```

然后让 OpenClaw 在 Python 沙箱中调用 AKShare：

```text
用 akshare 获取今天 A 股涨幅前 10 的股票，
显示股票名称、代码、涨幅、成交量、换手率。
```

常用接口速查：

| 接口 | 用途 |
|------|------|
| `ak.stock_zh_a_spot_em()` | 全部 A 股实时行情 |
| `ak.stock_zh_a_hist()` | 个股历史日线数据 |
| `ak.stock_board_industry_name_em()` | 行业板块列表 |
| `ak.stock_hsgt_north_net_flow_in_em()` | 北向资金净流入 |
| `ak.stock_market_activity_legu()` | 涨跌家数/涨停跌停统计 |

### 设置每日自动推送

配置定时任务，工作日自动推送盘前简报和盘后复盘：

```text
帮我创建两个定时任务：

1. 盘前简报：每个工作日早上 8:30（Asia/Shanghai），
   整理隔夜美股三大指数涨跌、A50 期货走势、今日重要事件日历，
   发送到飞书。

2. 盘后复盘：每个工作日下午 3:30（Asia/Shanghai），
   整理今日大盘涨跌、成交量变化、板块涨跌幅 TOP5、
   北向资金流向、我的自选股表现，
   发送到飞书。

自选股列表：600519 贵州茅台、300750 宁德时代、000858 五粮液。
```

## 实用建议

- **AKShare 是首选数据源**：免费、开源、接口全面。如果需要更高频数据（分钟级），可申请 Tushare Pro 积分（通过环境变量 `TUSHARE_TOKEN` 传入）
- **MCP 方案更省心**：不需要自己写 Python，OpenClaw 通过 MCP 协议自动调用数据接口
- **云端部署保持在线**：如果需要每天准时推送，推荐部署到云服务器（阿里云/腾讯云轻量服务器，月费几十元），配合 cron 实现全自动
- **不要依赖 AI 做交易决策**：当前 LLM 在金融决策上存在延迟、不稳定、不可解释等问题。把 OpenClaw 当作信息收集和整理工具，决策留给自己
- **注意接口频率**：AKShare 底层依赖公开网站数据，高频调用可能被限速。日常监控（每天几次）完全没问题

> **⚠️ 免责声明**：本用例仅用于个人投资信息整理，不构成投资建议。股市有风险，投资需谨慎。

## 相关链接

- [AKShare 官方文档](https://akshare.akfamily.xyz/) — 免费开源 A 股数据接口（16,000+ stars）
- [mcp-cn-a-stock - GitHub](https://github.com/elsejj/mcp-cn-a-stock) — A 股数据 MCP 服务（400+ stars）
- [Jason Zhang - 用 OpenClaw 做 A 股量化的真实感受](https://junxinzhang.com/openclaw-quant-trading-a-share/) — 最坦诚的实测报告
- [腾讯云 - Lighthouse 部署 OpenClaw 股市分析师](https://blog.csdn.net/u012263509/article/details/157762036) — 部署教程
- [阿里云 - 零门槛部署 OpenClaw 接入 A 股数据](https://developer.aliyun.com/article/1713167) — 部署教程
