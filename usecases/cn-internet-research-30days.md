# 中文互联网 30 天研究工具

> 与 [竞争对手分析与价格监控](competitive-intelligence.md) 互补：竞品分析偏重英文互联网和 SEO 数据，本用例专注中文互联网 8 大平台的社区真实讨论。

做产品决策、写竞品调研、追热点话题，你需要的不是搜索引擎排名靠前的页面，而是微博、知乎、小红书、B站等平台上真实用户在聊什么。手动逐个平台搜索、筛选、整理至少要半天。

这个用例让 AI Agent 一句话搜索中文互联网 8 大平台最近 30 天的内容，自动评分排序，生成带来源引用的研究报告。

## 它能做什么

- **8 平台并发搜索**：微博、小红书、B站、知乎、抖音、微信公众号、百度、今日头条，一次查询覆盖主流中文内容平台
- **三级降级策略**：API 优先 -> Playwright 爬虫 -> 公开接口，自动选择可用的数据获取方式，最大化可用性
- **综合评分排序**：相关性 45% + 时效性 25% + 互动度 30%（各平台互动指标不同：微博看转评赞、B站看播放弹幕投币、小红书看点赞收藏）
- **跨平台去重与交叉引用**：不同平台讨论同一话题时自动关联，避免重复
- **灵活输出**：compact（简报）、json（程序处理）、md（完整报告）、context（注入 Agent 上下文）多种格式
- **零配置可用**：安装 jieba 即可搜索 4 个免费平台（B站、知乎、百度、头条）；加装 Playwright 解锁 7/8 平台

## 能力边界

| 场景 | 可靠性 | 说明 |
|------|--------|------|
| 免费平台搜索（B站/知乎/百度/头条） | ✅ 可靠 | 公开 API，无需配置 |
| Playwright 爬虫模式（微博/小红书/抖音） | ⚠️ 基本可用 | 依赖浏览器自动化，平台反爬策略可能导致偶尔失败 |
| 微信公众号搜索 | ⚠️ 需 API Key | 唯一不支持爬虫降级的平台，无 Key 时回退到搜狗搜索 |
| 大规模高频采集 | ❌ 不适用 | 工具设计为按需研究，非持续监控；请遵守各平台服务条款 |

## 所需技能

[last30days-skill-cn](https://github.com/Jesseovo/last30days-skill-cn)（240+ stars，开源，见 LICENSE）—— 基于 [mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill)（22,000+ stars）的中文本土化版本，由 Jesse ([@Jesseovo](https://github.com/Jesseovo)) 开发维护。v2.0 集成 MediaCrawler 爬虫引擎思路，覆盖微博、小红书、B站、知乎、抖音、微信、百度、头条。

Python 依赖：

- `jieba`（必需）—— 中文分词，用于关键词提取和相关性计算
- `playwright`（推荐）—— 浏览器自动化，解锁爬虫模式

## 如何设置

### 方案一：零配置快速开始

安装技能和依赖：

```bash
# 安装技能（以 OpenClaw 为例）
git clone https://github.com/Jesseovo/last30days-skill-cn.git ~/.agents/skills/last30days-cn

# Claude Code 用户
# claude install Jesseovo/last30days-skill-cn

# 安装必需依赖
pip install jieba
```

验证安装，查看各平台可用状态：

```bash
python3 ~/.agents/skills/last30days-cn/scripts/last30days.py --diagnose
```

输出示例（零配置下 4 个平台可用）：

```json
{
  "bilibili": true,
  "zhihu": true,
  "baidu_api": false,
  "toutiao": true,
  "crawler_engine": {
    "playwright_available": false
  }
}
```

直接搜索试试：

```bash
python3 ~/.agents/skills/last30days-cn/scripts/last30days.py "AI编程助手" --emit compact
```

### 方案二：安装 Playwright 解锁 7/8 平台

```bash
pip install playwright
playwright install chromium
```

安装后微博、小红书、抖音、B站（备用）、知乎（备用）均可通过爬虫模式使用，无需 API Key。再次运行 `--diagnose` 确认 `playwright_available: true`。

### 方案三：配置 API Key（可选，更稳定）

仅在需要更稳定的数据获取时配置，所有 Key 都是可选的：

```bash
mkdir -p ~/.config/last30days-cn
touch ~/.config/last30days-cn/.env
chmod 600 ~/.config/last30days-cn/.env
```

编辑 `.env` 按需填入（全部可选）：

```ini
# 微博开放平台 - https://open.weibo.com
WEIBO_ACCESS_TOKEN=

# 抖音 - https://tikhub.io
TIKHUB_API_KEY=

# 微信公众号（唯一需要 Key 的平台）
WECHAT_API_KEY=

# 百度搜索 API - https://cloud.baidu.com
BAIDU_API_KEY=
BAIDU_SECRET_KEY=
```

### 在 Agent 中使用

安装后，直接用自然语言指令触发研究：

```text
搜索最近 30 天中文互联网上关于 "AI 编程工具" 的讨论，
覆盖微博、小红书、B站、知乎等平台，
生成一份带来源引用的研究报告。
```

指定平台和时间范围：

```text
只搜索 B站和知乎，最近 7 天关于 "Rust 语言" 的讨论。
```

深度研究模式（更多数据源、更长超时）：

```text
对 "新能源汽车" 做一次深度研究，覆盖所有平台，保存原始结果到 ~/Documents/research。
```

### CLI 参数速查

| 参数 | 说明 | 示例 |
|------|------|------|
| `--emit MODE` | 输出模式 | `compact` / `json` / `md` / `context` / `path` |
| `--quick` | 快速搜索（更少数据源） | 适合日常快速查询 |
| `--deep` | 深度搜索（更多数据源） | 适合正式调研 |
| `--days N` | 回溯天数（1-30） | `--days 7` 只看最近一周 |
| `--search SOURCES` | 指定搜索源 | `--search weibo,bilibili,zhihu` |
| `--save-dir DIR` | 保存原始输出 | `--save-dir ~/Documents/research` |
| `--diagnose` | 诊断各平台可用状态 | 排查问题时使用 |

## 实用建议

- **零配置先跑起来**：B站、知乎、百度、头条四个平台不需要任何配置，先用这四个验证效果，再考虑扩展
- **Playwright 是性价比最高的升级**：`pip install playwright && playwright install chromium` 两条命令就能解锁微博、小红书、抖音三个高价值平台，无需申请任何 API Key
- **控制请求频率**：爬虫模式模拟正常浏览行为，但高频调用仍有被封禁风险。日常研究（每天几次）完全没问题，不要用于持续监控
- **`--quick` 日常够用**：默认模式全局超时 180 秒，`--quick` 只需 90 秒。平时快速查询用 `--quick`，正式调研再用 `--deep`
- **结合 Agent 上下文使用**：`--emit context` 模式会输出适合注入 Agent 上下文的精简格式，让 Agent 基于最新的中文互联网讨论来回答问题或生成内容，而不是依赖过时的训练数据
- **注意法律合规**：爬虫功能仅供学习研究，严禁商业用途和大规模数据采集。使用前请阅读项目的免责声明，遵守各平台服务条款和中国相关法律法规

## 相关链接

- [last30days-skill-cn - GitHub](https://github.com/Jesseovo/last30days-skill-cn) — 中文互联网 8 大平台研究引擎（240+ stars）
- [last30days-skill - GitHub](https://github.com/mvanhorn/last30days-skill) — 原版英文项目，覆盖 Reddit/X/YouTube/HN 等（20,000+ stars）
- [汇智网 - /last30days 趋势研究技能](https://www.hubwiz.com/blog/last30days-trend-research-skill/) — 原版工具中文介绍
- [MediaCrawler - GitHub](https://github.com/NanmiCoder/MediaCrawler) — v2.0 爬虫引擎的技术灵感来源
