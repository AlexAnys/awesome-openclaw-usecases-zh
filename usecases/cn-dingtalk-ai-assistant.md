# 钉钉 AI 助手

钉钉是很多中小企业的主力办公工具，但内置的 AI 能力有限，很多场景覆盖不到。你想让 AI 帮你整理邮件、查资料、写文档，但钉钉自带的功能做不到，又不想让团队成员学新工具。

这个用例把 OpenClaw 部署为钉钉机器人。在钉钉对话中发消息就能触发 AI 任务，支持 Stream 模式（无需公网 IP），个人电脑即可运行。

## 它能做什么

- **对话式 AI 助手**：在钉钉私聊或群聊中直接与 OpenClaw 对话
- **Stream 模式**：WebSocket 长连接，无需公网 IP 或域名
- **多媒体支持**：支持图片、语音、视频、文件的接收和处理（部分出站类型依赖插件版本）
- **AI 卡片流式输出**：回复以钉钉卡片形式实时流式显示
- **Markdown 回复**：支持格式化的 Markdown 消息
- **群聊 @触发**：群聊中 @机器人才响应，不打扰正常沟通

## 所需技能

[openclaw-channel-dingtalk](https://github.com/soimy/openclaw-channel-dingtalk)（@soimy/dingtalk）—— 社区维护的钉钉通道插件

## 如何设置

### 第一步：创建钉钉应用

在 [钉钉开放平台](https://open-dev.dingtalk.com) 创建企业内部应用，开启机器人能力。

### 第二步：获取凭证

在应用信息页面记录 Client ID（AppKey）和 Client Secret（AppSecret）。

### 第三步：配置消息接收模式

**关键**：选择"Stream 模式"——这样不需要公网 IP 或域名，个人电脑就能运行（与飞书的长连接模式类似）。

### 第四步：安装插件并配置

```bash
openclaw plugins install @soimy/dingtalk
```

安装后需将插件加入安全白名单。编辑 `~/.openclaw/openclaw.json`，添加：

```json
{
  "plugins": {
    "enabled": true,
    "allow": ["@soimy/dingtalk"]
  }
}
```

通过 `openclaw onboard` 交互式引导或手动编辑 `~/.openclaw/openclaw.json` 的 `channels` 配置，填入 Client ID 和 Client Secret。

### 第五步：启动并测试

```bash
openclaw gateway restart
```

在钉钉中搜索你的机器人，发送消息测试。确认正常后设置开机自启：

```bash
openclaw gateway install
```

## 实用建议

- **Stream 模式是首选**：和飞书一样，不需要公网 IP，个人电脑或 NAS 即可运行
- **选飞书还是钉钉？按团队实际使用的 IM 选**：哪个是你们每天打开的工具，就接哪个。两个都用的团队可以同时接入
- **群聊策略**：建议设为"@机器人时才回复"，避免群聊中过于活跃
- **安全策略**：开启 pairing/allowlist，限制谁可以使用机器人的高级功能

## 相关链接

- [openclaw-channel-dingtalk - GitHub](https://github.com/soimy/openclaw-channel-dingtalk)
- [腾讯云 - 保姆级教程：OpenClaw 接入钉钉](https://cloud.tencent.com/developer/article/2625121)
- [腾讯云 - 快速接入指南](https://cloud.tencent.com/developer/article/2626553)
- [CSDN - 钉钉接入 OpenClaw 完整指南](https://blog.csdn.net/weixin_42125125/article/details/158430832)
- [阿里云 - 预装镜像方案](https://help.aliyun.com/zh/simple-application-server/use-cases/quickly-deploy-and-use-openclaw)
