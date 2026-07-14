# 版本更新与 Changelog 监控

> 含国内适配：Gitee / 钉钉 / 飞书 / 企业微信 webhook 推送

追踪你依赖的工具、应用和服务的版本更新——再也不会错过新功能、安全补丁或破坏性变更。

我们每个人在工作和生活中都依赖几十种软件工具：开发框架、生产力应用、SaaS 服务、移动应用。但保持更新意味着手动检查网站、阅读新闻通讯，或者只有在出了问题时才发现版本变更。重要的安全补丁和有用的功能可能被忽略几个月之久。

## 它能做什么

- 从多个来源监控版本更新：
  - GitHub releases（开源工具发布）
  - 包管理器（npm、PyPI、Homebrew）
  - 官方 changelog（变更日志）和 RSS 订阅源
  - App Store 或应用商店更新
- 在本地记录上次已知版本，避免重复通知
- 仅在实际版本变更时发出提醒，而不是每次检查都通知
- 推送简洁的摘要，包含版本差异、发布日期和关键变更

## 所需技能

- 自定义监控脚本，需实现：
  - 从各种 API 或网页来源获取最新版本
  - 在本地文件中存储状态（JSON、SQLite 等）
  - 格式化通知，包含版本对比和重点变更

## 如何设置

1. 为你的特定来源创建监控脚本（GitHub API、RSS 解析器等）
2. 添加需要追踪的工具/服务及其更新来源
3. 设置每日或每周定时任务：

```text
Run version monitor every morning at 9am. Notify me only when there are actual updates.
```

> 中文说明：每天早上 9 点运行版本监控。仅在有实际更新时通知我。

## 提示词

**立即检查更新：**

```text
Run version monitor now and show me any updates across all tracked tools.
```

> 中文说明：立即运行版本监控，显示所有追踪工具的更新。

**添加新的追踪工具：**

```text
Add VS Code to my version monitor. Check the official site for updates.
```

> 中文说明：将 VS Code 添加到版本监控中，检查官方站点获取更新。

**查看当前追踪的版本：**

```text
Show me all the versions I'm currently tracking and when they were last checked.
```

> 中文说明：显示我当前追踪的所有版本及其上次检查时间。

## 实用建议

- **从最重要的工具开始**：不要一次追踪太多，先把你每天依赖的 5-10 个核心工具加入监控
- **区分重要性**：安全补丁需要立即通知，功能更新可以汇总为每周摘要
- **配合 RSS 使用**：很多项目的 GitHub releases 页面自带 RSS 订阅源（在 URL 后加 `.atom`），无需编写 API 调用
- **版本号对比**：注意语义化版本（Semantic Versioning）——主版本号升级可能包含破坏性变更，需要重点关注

## 中国用户适配

国内开发者的工具生态与海外有一定差异：许多国产开源项目托管在 Gitee 而非 GitHub；团队协作使用钉钉、飞书、企业微信而非 Slack/Discord。以下方案解决国内场景下的版本监控和通知推送问题。

### Gitee 发行版监控

Gitee 提供与 GitHub 类似的 releases（发行版）功能和 API，可用于监控国内开源项目的版本更新。

**API 端点**：

```
GET https://gitee.com/api/v5/repos/{owner}/{repo}/releases
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `access_token` | 用户授权令牌（可选，公开仓库无需） |
| `page` | 页码 |
| `per_page` | 每页数量 |
| `direction` | 排序方向，`desc` 为降序 |

**示例：获取 LLaMA-Factory 最新发行版**：

```bash
curl -s "https://gitee.com/api/v5/repos/hiyouga/LLaMA-Factory/releases?per_page=1&direction=desc"
```

返回结果中的 `tag_name` 为版本号，`body` 为变更说明，`created_at` 为发布时间。

**GitHub + Gitee 双源监控脚本思路**：

```text
Set up a version monitor that checks both GitHub and Gitee releases:
- GitHub: langchain-ai/langchain, vllm-project/vllm, ollama/ollama
- Gitee: hiyouga/LLaMA-Factory
- GitHub: PaddlePaddle/Paddle

Store last-known versions in ~/version-monitor/state.json.
Check every morning at 9am. Only notify when versions change.
```

> 中文说明：设置同时检查 GitHub 和 Gitee 发行版的版本监控。在本地存储已知版本，每天早上 9 点检查，仅在版本变更时通知。

### 钉钉机器人推送

在钉钉群聊中添加自定义机器人，获取 webhook 地址后即可推送版本更新通知。

**创建步骤**：群设置 -> 智能群助手 -> 添加机器人 -> 自定义（通过 Webhook 接入）-> 复制 Webhook 地址。

**安全设置**（三选一）：
- 自定义关键词：消息中包含指定关键词才会推送（推荐设置为"版本更新"）
- 加签：使用密钥对请求签名
- IP 白名单：限制请求来源 IP

**推送示例**：

```bash
curl -X POST "https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "markdown",
    "markdown": {
      "title": "版本更新通知",
      "text": "## 版本更新通知\n\n| 项目 | 旧版本 | 新版本 |\n|------|--------|--------|\n| LangChain | 0.3.1 | 0.3.2 |\n| vLLM | 0.6.0 | 0.6.1 |\n\n> 检测时间：2026-03-11 09:00"
    }
  }'
```

> **安全提醒**：Webhook 地址和 access_token 属于敏感信息，请通过环境变量配置（如 `DINGTALK_WEBHOOK_URL`），不要硬编码在脚本中或提交到代码仓库。

### 飞书机器人推送

在飞书群聊中创建自定义机器人，通过 webhook 地址推送版本更新消息。

**创建步骤**：群设置 -> 群机器人 -> 添加机器人 -> 自定义机器人 -> 复制 Webhook 地址。

**安全校验**（可选）：支持签名校验，通过密钥对请求进行验证。

**推送示例**：

```bash
curl -X POST "https://open.feishu.cn/open-apis/bot/v2/hook/YOUR_HOOK_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "interactive",
    "card": {
      "header": {
        "title": { "tag": "plain_text", "content": "版本更新通知" },
        "template": "blue"
      },
      "elements": [
        {
          "tag": "markdown",
          "content": "**LangChain** `0.3.1` -> `0.3.2`\n**vLLM** `0.6.0` -> `0.6.1`\n\n检测时间：2026-03-11 09:00"
        }
      ]
    }
  }'
```

> **安全提醒**：Webhook 地址通过环境变量配置（如 `FEISHU_WEBHOOK_URL`），不要在公开仓库中暴露。

### 企业微信机器人推送

在企业微信群聊中添加群机器人，获取 webhook 地址后推送版本更新。

**创建步骤**：群聊设置 -> 群机器人 -> 添加机器人 -> 新创建一个机器人 -> 复制 Webhook 地址。

**推送示例**：

```bash
curl -X POST "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "markdown",
    "markdown": {
      "content": "## 版本更新通知\n\n> 项目 | 旧版本 | 新版本\n> ---|---|---\n> LangChain | 0.3.1 | 0.3.2\n> vLLM | 0.6.0 | 0.6.1\n\n检测时间：2026-03-11 09:00"
    }
  }'
```

> **安全提醒**：Webhook 地址通过环境变量配置（如 `WECOM_WEBHOOK_URL`），避免泄露导致垃圾消息。

### 国内开发者推荐监控清单

以下是国内开发者常用的工具和框架，建议加入版本监控：

| 项目 | 来源 | 说明 |
|------|------|------|
| [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) | GitHub / Gitee | 大模型微调框架，更新频繁 |
| [vLLM](https://github.com/vllm-project/vllm) | GitHub | 高性能推理引擎 |
| [LangChain](https://github.com/langchain-ai/langchain) | GitHub / PyPI | LLM 应用开发框架 |
| [Ollama](https://github.com/ollama/ollama) | GitHub | 本地大模型运行工具 |
| [Dify](https://github.com/langgenius/dify) | GitHub | LLM 应用开发平台 |
| [PaddlePaddle](https://github.com/PaddlePaddle/Paddle) | GitHub / PyPI | 百度深度学习框架 |
| [OpenClaw](https://github.com/openclaw/openclaw) | GitHub | 你正在使用的 AI 智能体平台 |
| [MindSpore](https://gitee.com/mindspore/mindspore) | Gitee | 华为 AI 计算框架 |
| [FastGPT](https://github.com/labring/FastGPT) | GitHub | 知识库问答平台 |
| [GLM-4](https://github.com/THUDM/GLM-4) | GitHub | 清华智谱开源大模型 |

**添加监控的提示词示例**：

```text
Set up version monitoring for my AI development stack:
- GitHub releases: hiyouga/LLaMA-Factory, vllm-project/vllm, langchain-ai/langchain, ollama/ollama, langgenius/dify
- Gitee releases: mindspore/mindspore
- GitHub releases (additional): PaddlePaddle/Paddle
- PyPI: langchain, transformers, torch

Check daily at 9am. Send updates to my DingTalk group via webhook.
Store state in ~/version-monitor/state.json.
```

> 中文说明：为 AI 开发技术栈设置版本监控，同时检查 GitHub 和 Gitee 上的发行版以及 PyPI 包。每天早上 9 点检查，通过钉钉 webhook 推送更新。

### 实用建议

- **Gitee 镜像仓库**：很多国际项目在 Gitee 上有官方镜像，可以直接使用 Gitee API 获取 releases，访问更快更稳定。注意部分项目可能未在 Gitee 发布 releases，使用前请先验证
- **RSS 替代方案**：如果 API 调用受限，可以使用 [RSSHub](https://docs.rsshub.app/) 生成 Gitee 仓库的 RSS 订阅源，具体路由格式请参考 [RSSHub 文档](https://docs.rsshub.app/)
- **多通道推送**：可以同时推送到钉钉（开发组）和飞书（管理层），不同群组关注不同项目
- **PyPI 国内镜像**：检查 Python 包版本时，可使用清华镜像源 `https://pypi.tuna.tsinghua.edu.cn/simple/` 加速访问

## 相关链接

- [GitHub API - Releases](https://docs.github.com/en/rest/releases/releases) — GitHub releases API 文档
- [Gitee API V5](https://gitee.com/api/v5/swagger) — Gitee 开放 API 文档
- [RSS Feed Parser libraries](https://github.com/topics/rss-parser) — RSS 解析库合集
- [RSSHub](https://docs.rsshub.app/) — 万物皆可 RSS，支持 Gitee releases 订阅
- [钉钉自定义机器人文档](https://open.dingtalk.com/document/orgapp/custom-robots-send-group-messages) — 钉钉 webhook 配置指南
- [飞书自定义机器人文档](https://open.feishu.cn/document/client-docs/bot-v3/add-custom-bot) — 飞书 webhook 配置指南
- [企业微信群机器人文档](https://developer.work.weixin.qq.com/document/path/91770) — 企业微信 webhook 配置指南

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/23)
