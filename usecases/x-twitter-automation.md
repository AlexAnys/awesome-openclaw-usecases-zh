# X/Twitter 全自动化

> 含国内适配：微博（现状说明）

通过自然语言在聊天中完成 X/Twitter 的全部操作——发推、回复、点赞、转发、关注、私信、搜索、数据提取、抽奖和账号监控。TweetClaw 插件将 OpenClaw 连接到 X/Twitter API，你无需切换任何第三方工具。

> **与 `x-account-analysis.md` 的关系**：该用例专注于 X 账号的**定性分析**（帖子质量、互动规律），而本用例覆盖的是**操作层**——发布、互动、数据提取和自动化监控。两者完全互补。

## 它能做什么

TweetClaw 是一个 OpenClaw 插件（plugin），通过托管 API（managed API）与 X/Twitter 交互，覆盖 40+ 端点（endpoint）：

- **发布与互动** — 发推、回复、点赞、转发、关注/取关、发送私信（DM）、更新个人资料和头像
- **搜索与提取** — 搜索推文和用户，提取关注者、点赞者、转发者、引用推文者、列表成员，导出为 CSV
- **抽奖** — 从推文互动中随机抽取获奖者，支持筛选条件（最低粉丝数、账号年龄、关键词）
- **监控** — 监控指定账号的新推文或粉丝变动，实时推送通知到聊天
- **内容创作** — 撰写、打磨、评分推文；管理草稿；分析写作风格
- **趋势** — 获取 X 热门话题和多源策划的趋势雷达

所有操作通过托管 API 完成——不使用浏览器 cookie（网页凭证），不需要爬虫，凭证不会暴露给 LLM（大语言模型）。

## 所需技能

| 技能 | 说明 |
|------|------|
| [@xquik/tweetclaw](https://github.com/Xquik-dev/tweetclaw) | TweetClaw 插件，通过 Xquik 平台连接 X/Twitter API |

难度：⭐⭐（需要注册 Xquik 账号并获取 API Key）

## 如何设置

### 1. 安装插件

```bash
openclaw plugins install @xquik/tweetclaw
```

### 2. 获取 API Key 并配置

前往 [xquik.com/account-manager](https://xquik.com/account-manager) 注册并获取 API Key，然后配置：

```bash
# 设置 API Key（不要直接写入明文，建议使用环境变量）
export XQUIK_API_KEY="xq_YOUR_KEY"
openclaw config set plugins.entries.tweetclaw.config.apiKey "$XQUIK_API_KEY"
```

### 3. 启用事件轮询（可选）

如果需要实时监控推送，启用轮询（polling）功能：

```bash
# 启用轮询，每 60 秒检查一次新事件
openclaw config set plugins.entries.tweetclaw.config.pollingEnabled true
openclaw config set plugins.entries.tweetclaw.config.pollingInterval 60
```

### 4. 开始使用

TweetClaw 使用两个核心工具：
- **`explore`**（免费）— 搜索 API 规格，查找可用端点，不产生实际调用
- **`tweetclaw`**（执行）— 发起经过认证的 API 调用，认证信息自动注入

以下是一些实用提示词（保留英文原文）：

发一条推文：
```text
Post a tweet: "Just shipped a new feature — try it out!"
```

从推文互动中抽奖（设置最低粉丝数过滤）：
```text
Pick 3 random winners from the retweeters of this tweet: https://x.com/username/status/123456789. Exclude accounts with fewer than 50 followers.
```

提取数据并导出为 CSV：
```text
Extract all users who liked this tweet and export as CSV: https://x.com/username/status/123456789
```

监控某个账号的新推文：
```text
Monitor @elonmusk and notify me whenever he posts a new tweet.
```

搜索特定话题的推文：
```text
Search tweets about AI agents from the last 24 hours.
```

查看快捷命令（无需 LLM）：

| 命令 | 说明 |
|------|------|
| `/xstatus` | 查看账号信息、订阅状态、用量 |
| `/xtrends` | 获取热门话题 |
| `/xtrends tech` | 按分类筛选热门话题 |

### 费用说明

| 层级 | 费用 | 包含功能 |
|------|------|---------|
| 免费 | $0 | 推文撰写与评分、写作风格分析、草稿管理、趋势雷达、账号管理 |
| 订阅 | $20/月 | 全部功能：发布/互动、搜索、数据提取、抽奖、监控、webhook（网络钩子） |

## 实用建议

### 平台风控提醒

> **重要**：X/Twitter 对自动化操作有严格的速率限制（rate limit）和反滥用机制，违规可能导致账号受限甚至永久封禁。

- **速率限制**：X API v2 对不同端点有不同的调用频率上限（例如发推 300 条/3 小时、点赞 1000 次/24 小时）。TweetClaw 的托管 API 会处理部分限流，但高频操作仍需注意
- **行为模式**：避免短时间内大量重复操作（如批量关注/取关），这类行为容易触发 X 的自动检测系统
- **内容合规**：自动发布的内容需遵守 X 的服务条款（ToS），避免垃圾信息和误导内容
- **账号隔离**：建议为自动化操作创建独立的 X 账号，避免主账号受影响
- **渐进式使用**：新账号从低频操作开始，逐步增加使用量，模拟正常用户行为模式

### 工作流建议

- 先用 `explore` 工具了解可用端点和参数，再执行实际操作
- 监控功能适合长期运行的场景（如竞品追踪、KOL 动态监控），配合 OpenClaw 的 cron job（定时任务）效果更好
- 数据提取结果导出为 CSV 后可以配合其他分析工具使用
- 抽奖功能设置合理的过滤条件（最低粉丝数、账号年龄）可以有效排除机器人账号

## 相关链接

- [TweetClaw GitHub 仓库](https://github.com/Xquik-dev/tweetclaw) — 插件源码与文档（MIT 许可）
- [Xquik 平台](https://xquik.com) — TweetClaw 背后的 X/Twitter 数据与自动化平台
- [X API v2 速率限制文档](https://developer.x.com/en/docs/x-api/rate-limits) — 官方速率限制参考

---

## 中国用户适配

### X/Twitter 访问

中国大陆用户访问 X/Twitter 需要网络代理工具，这是使用 TweetClaw 的前提条件。配置 OpenClaw 时需确保代理环境可用：

```bash
# 确保代理环境变量已设置（根据实际代理工具调整）
export HTTPS_PROXY="http://127.0.0.1:7890"
```

### 微博自动化现状

微博作为国内最接近 X/Twitter 的社交平台，理论上是本地化适配的首选。但实际情况如下：

**微博开放平台 API 准入门槛高**：
- 开发者需完成实名认证，创建应用并提交审核
- 部分高级 API（如发布微博、私信）仅对**蓝V/橙V 认证用户**开放
- 平台正在推进 V1 → V2 接口迁移，旧接口逐步关闭，生态不稳定
- 个人开发者获取写入权限（发布、互动）的审核流程复杂，通过率不确定

**缺乏成熟的 OpenClaw 集成方案**：
- 目前没有类似 TweetClaw 的成熟微博插件可直接安装
- 社区中有零散的微博 API 封装（如基于 `weibo` Python 包），但缺乏维护和 OpenClaw 生态适配
- 微博的反爬和风控机制比 X 更严格，非官方 API 方式风险高

**现实建议**：
- 如果你主要面向国际受众，直接使用 TweetClaw 操作 X/Twitter 是最佳方案
- 如果需要微博自动化，建议通过微博开放平台官方渠道申请 API 权限，但需准备好企业资质
- 可关注 [微博开放平台](https://open.weibo.com/) 的最新动态，等待生态改善

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/x-twitter-automation.md)
