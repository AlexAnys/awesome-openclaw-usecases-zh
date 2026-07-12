# 券商交易助手

> 含国内适配：富途证券 OpenAPI / 老虎证券 Open API / A 股券商 API 现状

TWS（Trader Workstation）是 Interactive Brokers 的桌面交易软件——一个 90 年代风格的 Java 巨无霸应用，界面密密麻麻，光找个下单窗口就要点好几层菜单。如果你想让 AI 帮你查持仓、看盈亏、预览订单，必须先绕过这个笨重的 GUI。

ibkr-cli 把 IBKR 的交易能力搬到了命令行，让 OpenClaw 能直接通过终端命令操作你的券商账户。最关键的安全设计：**所有交易指令默认只预览，必须显式加 `--submit` 才会真实执行**。

> **⚠️ 免责声明**：本用例仅用于个人投资信息查询和订单管理的自动化，不构成投资建议。证券交易有风险，投资需谨慎。使用 AI 辅助交易请务必仔细验证每一笔订单，最终决策权和责任在你自己。

## 它能做什么

- **账户总览**：查看账户摘要（净值、购买力、保证金）和持仓明细
- **订单管理**：查询挂单/成交/执行记录，取消或修改订单
- **安全下单**：预览订单的预估成本和影响，确认无误后才提交
- **多种订单类型**：市价单、限价单、止损单、止损限价单、追踪止损、括号单（止盈+止损）
- **实时行情**：个股报价快照、定时刷新、历史 K 线数据
- **新闻资讯**：按标的获取新闻标题和全文
- **期权数据**：期权链查询（到期日、行权价、Greeks）
- **市场扫描**：按涨幅、成交量、股息率等条件筛选股票
- **公司基本面**：财务报表、关键指标、机构持仓
- **历史记录**：交易记录、盈亏明细、资金出入、股息记录（通过 Flex Queries）

## 能力边界

| 场景 | 可靠性 | 说明 |
|------|--------|------|
| 持仓/账户查询 | ✅ 可靠 | 直连 IBKR API，数据实时准确 |
| 订单预览 | ✅ 可靠 | what-if 模式预估成本，不触发真实交易 |
| 订单提交 | ✅ 可靠 | 需显式 `--submit`，支持多种订单类型 |
| 行情/基本面 | ✅ 可靠 | 依赖 IBKR 市场数据订阅，延迟行情自动回退 |
| AI 交易决策 | ❌ 不推荐依赖 | LLM 不具备金融决策能力，仅用于信息整理和操作执行 |

## 所需技能

**核心工具**：

[ibkr-cli](https://github.com/fatwang2/ibkr-cli)（MIT 开源）—— 基于 `ib_async`、`Typer`、`Rich` 构建的 Interactive Brokers 命令行工具，提供 OpenClaw skill 一键安装。

**前置条件**：

- 一个 Interactive Brokers 账户（支持模拟盘和实盘）
- 运行中的 IB Gateway 或 TWS（本地连接，无需公网暴露）
- Python 3.10+

## 如何设置

### 第一步：安装 ibkr-cli skill

在 OpenClaw 中安装 skill，让 Agent 自动引导你完成所有后续步骤：

```bash
npx skills add fatwang2/ibkr-cli
```

安装后，直接告诉 OpenClaw 你想做什么：

```text
帮我安装 ibkr-cli 并连接我的 IBKR 账户。我已经有 IB Gateway 在运行了。
```

Agent 会自动引导你完成安装、配置 profile、验证连接。

### 第二步（手动安装，可选）：安装 CLI 工具

如果你更倾向于手动安装：

```bash
pipx install ibkr-cli
```

验证安装：

```bash
ibkr --help
ibkr --version
```

### 第三步：配置和连接

ibkr-cli 首次运行时自动创建配置文件，包含四个默认 profile：

| Profile | 用途 | 端口 |
|---------|------|------|
| `gateway-live` | IB Gateway 实盘 | 4001 |
| `gateway-paper` | IB Gateway 模拟盘 | 4002 |
| `live` | TWS 实盘 | 7496 |
| `paper` | TWS 模拟盘 | 7497 |

验证连接（建议先用模拟盘）：

```bash
ibkr doctor --profile gateway-paper
ibkr connect test --profile gateway-paper
```

### 第四步：日常使用示例

查看账户和持仓：

```text
帮我看一下 IBKR 账户摘要和当前持仓。
```

对应 CLI 命令：

```bash
ibkr account summary --profile gateway-paper
ibkr positions --profile gateway-paper
```

预览一笔交易（不会实际执行）：

```text
我想买 10 股 AAPL，先帮我预览一下成本。
```

```bash
ibkr buy AAPL 10 --preview --profile gateway-paper
```

确认后提交（需要你明确同意）：

```bash
ibkr buy AAPL 10 --submit --profile gateway-live
```

括号单（同时设止盈止损）：

```bash
ibkr buy AAPL 10 --type LMT --limit 150.00 --take-profit 160.00 --stop-loss 140.00 --preview --profile gateway-live
```

查看今日成交和盈亏：

```bash
ibkr orders executions --profile gateway-paper
ibkr pnl --days 1
```

### JSON 输出用于自动化

所有命令支持 `--json` 输出，方便 OpenClaw 解析和处理：

```bash
ibkr quote AAPL --profile gateway-paper --json
ibkr positions --profile gateway-paper --json
ibkr buy AAPL 10 --preview --profile gateway-paper --json
```

错误也会返回结构化 JSON（含 `error.code` 和 `error.message`），便于 Agent 自动处理异常。

## 安全机制详解

ibkr-cli 的安全设计值得单独说明，这是它区别于其他交易工具的核心：

1. **预览优先**：`buy` 和 `sell` 命令**必须**指定 `--preview` 或 `--submit` 之一，没有默认行为，不存在"不小心下单"的可能
2. **本地连接**：CLI 通过本地端口连接 IB Gateway/TWS，不经过互联网，API 密钥不会暴露
3. **模拟盘优先**：建议所有新命令先在 `gateway-paper` profile 上验证
4. **单连接串行**：同一 profile 同时只能有一个 CLI 进程连接，避免并发冲突

## 实用建议

- **先用模拟盘**：所有新命令和提示词都应在 paper trading 环境中验证通过后，再切换到实盘
- **不要让 AI 自主决策交易**：ibkr-cli 适合作为"执行层"——你决定买什么、买多少，AI 帮你执行操作和整理信息。不要设置让 AI 自动决策并提交订单的工作流
- **市场数据订阅**：IBKR 的实时行情需要订阅对应的数据包（美股基础行情免费，其他市场按需订阅）。没有订阅时，ibkr-cli 会自动回退到延迟行情
- **Flex Queries 历史数据**：交易记录、盈亏、资金流水等历史数据通过 IBKR Flex Queries 获取，不需要 IB Gateway 在线，但数据延迟 T-1
- **项目尚处早期**：ibkr-cli 于 2026 年 3 月发布，迭代活跃（11 天内发布了 11 个版本），但社区使用反馈尚不多。建议关注 [GitHub Issues](https://github.com/fatwang2/ibkr-cli/issues) 了解已知问题

## 中国用户适配

### IBKR 对中国用户的可用性

Interactive Brokers（盈透证券）支持中国大陆居民开户，可交易美股、港股、欧股等全球市场。但需要注意：

- 开户需提供身份证明和地址证明，审核周期约 1-2 周
- 入金需通过银行跨境汇款（招行、工行等均支持）
- IB Gateway 从国内连接可能需要稳定的网络环境

ibkr-cli 的所有功能对中国 IBKR 用户完全适用。

### A 股券商 API 现状

国内 A 股券商目前**没有面向个人投资者的公开交易 API**。这是中美券商生态的根本差异：

| 维度 | IBKR（美股） | 国内 A 股券商 |
|------|------------|-------------|
| API 可用性 | 开放 API，个人可接入 | 无公开个人 API |
| 程序化交易 | 合规允许 | 需要机构资质 |
| 替代方案 | 直接用 ibkr-cli | 见下方富途/老虎方案 |

### 港美股替代：富途证券 / 老虎证券

如果你主要交易港美股，以下两家券商提供了面向个人开发者的 API：

**富途证券 OpenAPI**（[官方文档](https://openapi.futunn.com/)）

- 支持港股、美股、A 股通（部分）
- 提供 Python SDK（`futu-api`），pip 安装即用
- 需要下载富途牛牛客户端作为网关（类似 IB Gateway 的角色）
- 模拟交易免费，实盘交易需开通 OpenAPI 权限

```bash
pip install futu-api
```

**老虎证券 Open API**（[官方文档](https://quant.itigerup.com/)）

- 支持美股、港股、A 股（沪港通/深港通）
- 提供 Python SDK（`tigeropen`）
- 同样需要本地网关客户端
- 有模拟交易环境

```bash
pip install tigeropen
```

> **注意**：富途和老虎的 API 目前**没有现成的 OpenClaw skill**，需要自行编写 Python 脚本或封装为 MCP 服务。如果你有封装经验，这是一个很好的社区贡献机会。

### A 股行情查询替代

如果你只需要 A 股行情查询（不涉及交易），可以配合 [A 股每日行情监控](cn-a-share-monitor.md) 用例，使用 AKShare 获取免费数据。ibkr-cli 负责港美股交易操作，AKShare 负责 A 股信息采集，两者互补。

## 相关链接

- [ibkr-cli - GitHub](https://github.com/fatwang2/ibkr-cli) — MIT 开源，OpenClaw skill 一键安装
- [ibkr-cli - PyPI](https://pypi.org/project/ibkr-cli/) — `pipx install ibkr-cli`
- [Interactive Brokers 官网](https://www.interactivebrokers.com/) — 开户和 IB Gateway 下载
- [Show HN: ibkr-cli](https://news.ycombinator.com/item?id=47426030) — Hacker News 首发讨论
- [A 股每日行情监控](cn-a-share-monitor.md) — AKShare 数据源，与本用例互补
- [富途 OpenAPI 文档](https://openapi.futunn.com/) — 港美股交易 API 替代方案
- [老虎证券 Open API](https://quant.itigerup.com/) — 港美股交易 API 替代方案
