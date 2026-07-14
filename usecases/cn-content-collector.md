# 多平台内容采集 → 飞书多维表格

信息过载是知识工作者的日常：微信公众号、知乎、B 站、即刻、X/Twitter、Reddit、Hacker News——每天在七八个平台之间跳来跳去，看到好文章随手收藏，但收藏夹永远只进不出。更大的问题是：收藏了也找不到，因为内容散落在各个平台，没有统一的索引。

这个用例把 OpenClaw 变成一个"内容收藏机器人"。在飞书对话中丢一个链接，它自动抓取原文、AI 生成摘要和分类、去重后写入飞书多维表格。你的内容库从此有了结构化索引，支持筛选、搜索和回顾。

> **与飞书 AI 助手的区别**：[飞书 AI 助手](cn-feishu-ai-assistant.md)解决的是"在飞书对话中使用 AI"——聊天触发任务、操作文档、管理日历。本用例解决的是"跨平台内容的自动化采集和结构化存储"——侧重点是抓取、整理、入库。两者互补：先用本用例把内容收进多维表格，再用飞书 AI 助手在对话中查询和加工这些内容。

## 它能做什么

- **多平台采集**：一个链接触发，覆盖国内外 7+ 平台
- **微信公众号完美抓取**：基于 Scrapling 绕过微信反爬，提取完整正文
- **AI 智能整理**：自动生成摘要、分类、关键词，不需要手动打标签
- **飞书多维表格存储**：结构化字段，支持按分类/来源/时间筛选
- **原文云端备份**：长文转为 .md 文件上传飞书云空间，多维表格只存摘要
- **自动去重**：URL 标准化 + 本地缓存 + 文档内容双重检查，不会重复收藏
- **截图识别**：发送截图也能通过 OCR 提取链接并触发采集

## 平台采集范围

| 平台 | 抓取方式 | 内容类型 | 备注 |
|------|---------|---------|------|
| 微信公众号 | Scrapling（`web-content-fetcher`） | 图文全文 | 完美绕过微信反爬 |
| 知乎 | defuddle | 回答/专栏文章 | 部分内容需登录 |
| B 站 | defuddle | 标题 + 描述 | 视频类仅提取元数据 |
| 即刻 | defuddle | 动态/文章 | 网页版可能需登录 |
| X/Twitter | x-tweet-fetcher | 推文/长文 | 专用提取器 |
| Reddit | defuddle + JSON API | 帖子/评论 | 支持 `.json` 结构化接口 |
| Hacker News | defuddle + Firebase API | 帖子/评论 | 支持 Firebase 结构化接口 |
| 其他网站 | defuddle → baoyu-url-to-markdown | 通用网页 | 两级 fallback |

## 所需技能

[content-collector-skill](https://github.com/vigorX777/content-collector-skill)（MIT，168 stars） —— 社区开发的多平台内容采集技能，作者是"懂点儿AI"公众号。

依赖的子技能（按需安装）：
- `feishu-doc` / `feishu-bitable` —— 飞书文档和多维表格读写
- `defuddle` —— 通用网页内容提取
- `x-tweet-fetcher` —— X/Twitter 专用提取
- `web-content-fetcher` —— 微信公众号抓取（基于 Scrapling）
- `baoyu-url-to-markdown` —— 通用 fallback

## 如何设置

### 第一步：安装技能和依赖

```bash
# 复制到 OpenClaw skills 目录
cp -r content-collector ~/.openclaw/workspace/skills/

# 安装 Scrapling（微信公众号抓取需要）
pip install scrapling html2text
scrapling install
```

### 第二步：创建飞书多维表格

在飞书中创建一个多维表格，包含以下字段：

| 字段名 | 字段类型 | 说明 |
|-------|---------|------|
| 标题 | 文本 | 内容标题，支持搜索 |
| 来源 | 文本 | 作者/平台名称，支持筛选 |
| 分类 | 单选 | 工具推荐 / 技术教程 / 实战案例 / 产品想法 |
| 原文链接 | 超链接 | 原始 URL |
| 摘要内容 | 文本 | AI 生成的 3-5 句摘要 |
| 原文文件 | 超链接 | 飞书云空间中的 .md 全文备份 |
| 记录时间 | 创建时间 | 自动记录 |

### 第三步：配置环境变量

```bash
export FEISHU_BITABLE_APP_TOKEN="your_app_token"
export FEISHU_BITABLE_TABLE_ID="your_table_id"
```

`APP_TOKEN` 和 `TABLE_ID` 从多维表格的 URL 中获取：`https://my.feishu.cn/base/{APP_TOKEN}?table={TABLE_ID}`。

### 第四步：使用

在飞书对话中直接发送链接，技能自动触发：

```text
https://mp.weixin.qq.com/s/xxxxxxxx
```

也可以发送截图（OCR 识别后提取链接），或用自然语言：

```text
帮我收藏这篇文章：https://zhuanlan.zhihu.com/p/123456789
```

## 工作流程

```
丢链接 → 平台检测 → 去重检查 → 抓取正文 → AI 摘要+分类 → 写入多维表格 → 发通知
```

详细拆解：

1. **平台检测**：`extract_content.py` 根据 URL 域名识别平台，返回推荐的抓取技能和 CSS 选择器
2. **去重检查**：`deduplicate.py` 对 URL 做标准化（统一域名别名、去除追踪参数、展开短链接），然后查本地缓存和飞书文档
3. **内容抓取**：根据平台调用对应技能——微信用 Scrapling，X 用专用提取器，其他用 defuddle
4. **AI 整理**：LLM 从原文中提取标题、来源、分类、摘要
5. **存储**：长文先上传飞书云空间生成 .md 文件链接，再将摘要和元数据写入多维表格
6. **去重更新**：成功入库后将 URL 加入本地缓存

## 反爬与风控提醒

自动化采集涉及多个平台的内容抓取，使用前请注意：

- **微信公众号**：Scrapling 通过模拟浏览器环境绕过反爬，但高频抓取仍可能触发风控。建议控制频率，不要批量连续抓取
- **知乎/即刻**：部分内容需要登录态，未登录时可能只能获取摘要或被重定向到登录页
- **B 站**：视频内容只能提取标题和描述，无法获取视频本身或弹幕
- **X/Twitter**：依赖 `x-tweet-fetcher` 技能，需要该技能自身的认证配置
- **通用建议**：仅采集公开可访问的内容，遵守各平台的 robots.txt 和使用条款。内容用于个人知识管理，不要用于商业转载或批量分发

## 实用建议

- **分类可自定义**：默认的四个分类（工具推荐/技术教程/实战案例/产品想法）可以根据你的需求在多维表格中修改，AI 会自动适配
- **去重缓存有上限**：本地缓存最多保留 1000 条、30 天有效。超出后旧记录会被淘汰，但飞书文档中的记录不受影响
- **短链接自动展开**：支持 t.co、bit.ly 等常见短链接服务，展开后再做去重判断
- **原文备份很重要**：公众号文章可能被删除，上传到飞书云空间的 .md 文件是你的永久备份
- **配合飞书视图**：在多维表格中创建不同视图——按分类分组、按时间排序、按来源筛选——让内容库真正可用

## 相关链接

- [content-collector-skill GitHub 仓库](https://github.com/vigorX777/content-collector-skill) — 源码、安装说明、版本记录
- [从零打造 AI 内容收藏助手](https://mp.weixin.qq.com/s/hw4uKk-9ezaJlDpL1nEUuA) — 作者的制作过程分享（"懂点儿AI"公众号）
- [飞书 AI 助手用例](cn-feishu-ai-assistant.md) — 互补用例：在飞书对话中使用 AI 操作文档和任务
