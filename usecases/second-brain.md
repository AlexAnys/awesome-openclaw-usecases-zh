# 第二大脑

> 含国内适配：钉钉 / 飞书 / 企业微信

你会产生想法、发现有趣的链接、听说要读的书——但你从来没有一个好的系统来捕获它们。Notion 变得复杂，Apple Notes 变成了一个拥有 10,000 条未读笔记的坟场。你需要的是像给朋友发短信一样简单的东西。

这个工作流将 OpenClaw 变成一个通过短信交互的记忆捕获系统，背后有一个可随时浏览的自定义可搜索界面。

## 功能介绍

- 通过 Telegram、iMessage 或 Discord 向你的 OpenClaw 发送任何内容——"提醒我读一本关于本地 LLM 的书"——它会立即记住
- OpenClaw 内置的记忆系统会永久存储你告诉它的一切
- 自定义的 Next.js 仪表盘让你可以搜索每一条记忆、对话和笔记
- 全局搜索（Cmd+K）覆盖所有记忆、文档和任务
- 无需文件夹、无需标签、无需复杂的组织——只需输入和搜索

## 痛点

每个笔记应用最终都会变成负担。你停止使用它，因为组织的摩擦比遗忘的摩擦更高。关键洞察是：**捕获应该像发短信一样简单，检索应该像搜索一样简单**。

## 所需技能

- Telegram、iMessage 或 Discord 集成（用于基于文本的捕获）
- Next.js（OpenClaw 会为你构建——无需编码）

## 设置方法

1. 确保你的 OpenClaw 已连接到你偏好的消息平台（Telegram、Discord 等）。

2. 立即开始使用——只需向你的机器人发送任何你想记住的内容：

以下是一些记忆捕获的示例：

```text
Hey, remind me to read "Designing Data-Intensive Applications"
Save this link: https://example.com/interesting-article
Remember: John recommended the restaurant on 5th street
```

3. 通过提示 OpenClaw 构建可搜索的界面：

以下提示词让智能体为你构建一个完整的第二大脑 Web 应用：

```text
I want to build a second brain system where I can review all our notes,
conversations, and memories. Please build that out with Next.js.

Include:
- A searchable list of all memories and conversations
- Global search (Cmd+K) across everything
- Ability to filter by date and type
- Clean, minimal UI
```

4. OpenClaw 会为你构建并部署整个 Next.js 应用。访问它提供的 URL，你就拥有了自己的第二大脑仪表盘。

5. 从现在开始，每当你想到什么——在路上、在会议中、睡前——只需给你的机器人发消息。需要查找什么的时候再回到仪表盘。

## 关键洞察

- 力量在于**零摩擦捕获**。你不需要打开应用、选择文件夹或添加标签。只需发消息。
- OpenClaw 的记忆系统是累积的——它记住你*曾经*告诉它的一切，使其随时间变得更加强大。
- 你可以从手机给机器人发消息，它会在你的电脑上构建东西。界面就是对话本身。

## 参考来源

灵感来自 [Alex Finn 关于改变生活的 OpenClaw 用例的视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)。

## 相关链接

- [OpenClaw 记忆系统](https://github.com/openclaw/openclaw)
- [打造第二大脑（Tiago Forte）](https://www.buildingasecondbrain.com/)

## 中国用户适配

原用例使用 Telegram 作为消息捕获端，国内用户无法直接使用。以下是经过社区验证的国内 IM 替代方案。

### 方案对比

| 方案 | 门槛 | 公网 IP 需求 | OpenClaw 支持 | 推荐度 |
|------|------|-------------|--------------|--------|
| 钉钉 Stream | 最低 | 不需要 | 已验证 | 最高 |
| 飞书 | 中等 | 需要（或长连接） | 已验证 | 高 |
| 企业微信 | 中等 | 需要 | 已验证 | 高 |
| 个人微信 | — | — | — | 不推荐 |

### 钉钉 Stream 机器人（推荐首选）

零公网 IP、零域名、零证书、零防火墙白名单——个人钉钉账号即可，完全免费。

OpenClaw 生态已验证：`OpenClaw-Docker-CN-IM` 和 `openclaw-china` 均预装钉钉插件。

**参考资源**：
- GitHub: [ytanck/dingtalk-stream](https://github.com/ytanck/dingtalk-stream)
- 阿里云百炼平台有"10 分钟搭建钉钉 AI 机器人"教程

### 飞书机器人

支持事件订阅（`im.message.receive_v1`），需在开发者后台创建企业自建应用。OpenClaw 集成成熟，阿里云有官方教程"OpenClaw + 飞书集成"。

适合已使用飞书的团队。

### 企业微信自建应用

个人可注册（选"个人团队"类型），但未认证有功能限制。

**参考资源**：
- GitHub: [sunnoy/openclaw-plugin-wecom](https://github.com/sunnoy/openclaw-plugin-wecom)
- GitHub: [dingxiang-me/OpenClaw-Wechat](https://github.com/dingxiang-me/OpenClaw-Wechat)

### 一键部署方案

如果不想逐个配置插件，可以使用社区整合的一键部署镜像：

- [OpenClaw-Docker-CN-IM](https://github.com/justlovemaki/OpenClaw-Docker-CN-IM)：Docker 一键部署，预装飞书/钉钉/企微/QQ 插件
- [openclaw-china](https://github.com/BytePioneer-AI/openclaw-china)：OpenClaw 中国插件集合

### 关于个人微信

**明确不推荐**。Hook 方案封号率极高（>80%），Wechaty iPad 协议存在法律风险。这是项目红线。

如果你希望在微信生态内实现类似"第二大脑"功能，可以了解腾讯 IMA 知识库（非 OpenClaw 方案），它是微信生态内的零门槛知识管理产品，但不具备 OpenClaw 的智能体能力。

> **安全提示**：使用社区 skill 和第三方插件时，请自行审查代码和权限范围。IM 机器人的 API 凭证属于敏感信息，请通过环境变量或 `.env` 文件配置，确保 `.env` 已加入 `.gitignore`。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/second-brain.md)
