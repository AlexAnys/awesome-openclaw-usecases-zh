# 企业微信 AI 助手

微信是中国最常用的沟通工具，但 AI 能力无法直接在微信生态中使用——你得切到其他 App 用 AI，再手动把结果搬回微信。

这个用例把 OpenClaw 部署为企业微信应用。在企业微信里发消息就能触发 AI 任务，而且通过企业微信的"微信插件"功能，个人微信用户扫码关联后，也能直接在微信里和 AI 对话。

## 它能做什么

- **企业微信内对话式 AI**：在企业微信私聊或群聊中直接与 OpenClaw 对话
- **个人微信也能用**：通过企业微信的"微信插件"，关联后个人微信用户也能和 AI 对话
- **流式输出**：AI 回复实时逐字显示
- **群聊 AI 助手**：支持 @触发、指令白名单等策略
- **多媒体支持**：支持图片、文件的收发和处理

## 两种集成方案

| 方案 | 插件 | 特点 |
|------|------|------|
| openclaw-china 套件 | `@openclaw-china/wecom-app` | 社区维护的中国 IM 插件集合，一站式配置 |
| sunnoy 插件 | `@sunnoy/wecom` | 功能更丰富，支持动态 Agent 管理、群聊集成、指令白名单 |

两种方案都经过社区验证，选哪个看你的需求：只做基础对话选前者，需要群聊管理等高级功能选后者。

## 所需技能

- OpenClaw 2026.2.9+ 版本
- 企业微信管理员权限（创建自建应用）
- 公网 IP 或内网穿透工具（用于接收回调）

## 如何设置

### 第一步：创建企业微信应用

在 [企业微信管理后台](https://work.weixin.qq.com) → 应用管理 → 创建应用。记录 Corp ID、Corp Secret、Agent ID。

### 第二步：配置回调 URL

在应用设置中配置"接收消息"的回调 URL，格式一般为：

```
http://<你的公网IP>:18789/wecom-app
```

如果没有公网 IP，可以使用内网穿透工具（如 frp、ngrok）。

### 第三步：安装插件并配置

**方案 A：openclaw-china 套件**

```bash
openclaw plugins install @openclaw-china/wecom-app
```

在 `openclaw.json` 中配置：

```json
{
  "channels": {
    "wecom-app": {
      "enabled": true,
      "webhookPath": "/wecom-app",
      "token": "<企业微信 Token>",
      "encodingAESKey": "<企业微信 AES Key>",
      "corpId": "<Corp ID>",
      "corpSecret": "<Corp Secret>",
      "agentId": "<Agent ID>"
    }
  }
}
```

> **安全提醒**：请勿将真实凭证直接写入配置文件并提交到版本控制。建议使用环境变量：将 Token、AES Key 等敏感信息存入 `.env` 文件，并确保 `.env` 已加入 `.gitignore`。

**方案 B：sunnoy 插件**

```bash
openclaw plugins install @sunnoy/wecom
```

按 [插件文档](https://github.com/sunnoy/openclaw-plugin-wecom) 配置 channels。该插件支持 Bot 模式和 App 模式，具体配置参数请参考仓库 README。

> 安装第三方社区插件后，建议在 `plugins.allow` 中配置白名单（参考钉钉用例中的说明）。

### 第四步：启动并测试

```bash
openclaw gateway restart
```

在企业微信中找到你的应用，发一条消息测试。

### 第五步（可选）：关联个人微信

在企业微信管理后台 → 微信插件 → 邀请成员关联个人微信。关联后，成员可以在个人微信中直接和 AI 对话。

## 实用建议

- **强烈推荐企业微信路线**：使用官方 API，零封号风险，稳定可靠。不建议使用个人微信第三方协议方案（随时面临封号）
- **内网穿透**：与飞书的长连接模式不同，企业微信需要公网可达的回调 URL。如果你在家里部署，需要配置内网穿透
- **安全策略**：建议开启 pairing/allowlist，群聊设为 @触发，避免任何人都能使用
- **微信插件是杀手级功能**：它让你不需要让所有人都安装企业微信，个人微信关联后就能直接用

## 相关链接

- [openclaw-china 企业微信插件](https://github.com/BytePioneer-AI/openclaw-china)
- [sunnoy 企业微信插件（高级功能）](https://github.com/sunnoy/openclaw-plugin-wecom)
- [阿里云 - 企业微信接入 OpenClaw 官方教程](https://help.aliyun.com/zh/simple-application-server/use-cases/openclaw-enterprise-wechat-integration)
- [腾讯云 - 企业微信集成教程](https://cloud.tencent.com/developer/article/2625147)
- [阿里云开发者社区 - 2026 保姆级教程](https://developer.aliyun.com/article/1711514)
