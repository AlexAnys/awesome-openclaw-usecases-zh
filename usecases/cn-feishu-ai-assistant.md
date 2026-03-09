# 飞书 AI 助手

飞书是很多团队的主要沟通工具，但 AI 能力和日常工作流之间是割裂的——你需要切出飞书去用 AI，用完再切回来粘贴结果。

这个用例把 OpenClaw 直接部署为飞书机器人。在飞书对话中发消息就能触发 AI 任务，结果直接回复到对话里，不需要切换任何工具。

> **2026.3 更新**：飞书官方推出了 OpenClaw 飞书插件，支持以用户身份操作文档、日历、任务等。目前飞书接入 OpenClaw 有两种主流方式可选，详见下方[选择建议](#选择建议两种插件怎么选)。

## 它能做什么

- **对话式 AI 助手**：在飞书私聊或群聊中直接与 OpenClaw 对话
- **任务自动化**：通过聊天消息触发文件处理、信息查询、定时提醒
- **富媒体支持**：支持图片、文件的收发和处理
- **流式输出**：AI 回复实时逐字显示，不用等整段生成完
- **群聊管理**：可配置 @提及触发、白名单/黑名单等策略
- **多智能体路由**：一个飞书机器人可以根据用户/群组路由到不同的 AI 智能体

## 所需技能

基础消息功能无需额外安装——OpenClaw 2026.2+ 已内置飞书渠道插件。如果需要 AI 操作飞书文档、日历、任务等，可改用飞书官方插件（两者二选一，不能同时启用，见下方选择建议）。

## 选择建议：两种插件怎么选

| 维度 | 飞书官方插件 | OpenClaw 内置插件 |
|---|---|---|
| **操作身份** | 以**你本人**身份（OAuth 授权） | 以**机器人**身份 |
| **消息收发** | ✅ | ✅ |
| **文档操作** | ✅ 创建 + 编辑 + 读取 | 读取为主 |
| **多维表格** | ✅ 完整操作 | 基础读写 |
| **日历 / 任务** | ✅ | ❌ |
| **安装复杂度** | 需额外安装 CLI 工具 + OAuth | 内置即用，`openclaw channels add` |
| **维护方** | 飞书团队 | OpenClaw 社区 |

**简单说**：主要用飞书做聊天入口 → 选内置插件（简单）；需要 AI 帮你操作文档、建表、约会议 → 选飞书官方插件（功能强）。两者不能同时启用。

详细对比和迁移指南见 [openclaw-feishu 社区指南](https://github.com/AlexAnys/openclaw-feishu)。

## 如何设置

> 以下步骤适用于 **OpenClaw 内置插件**（方案 b）。如果选择飞书官方插件（方案 a），请参考 [飞书官方插件安装指南](https://github.com/AlexAnys/openclaw-feishu/blob/main/docs/feishu-official-plugin.md)。

完整的保姆级教程请参考 [openclaw-feishu 配置指南](https://github.com/AlexAnys/openclaw-feishu)，以下是核心步骤概要（约 15-20 分钟）：

### 第一步：创建飞书应用

在 [飞书开发者后台](https://open.feishu.cn) 创建企业自建应用，开启机器人能力。

### 第二步：配置权限

在"权限管理"中批量导入所需权限（`im:message`、`im:chat`、`im:resource` 等），教程中提供了完整的权限 JSON 可以一键导入。

### 第三步：配置事件订阅

**关键**：选择"使用长连接接收事件"模式——这样不需要公网 IP 或域名，你的个人电脑就能运行。

添加事件：`im.message.receive_v1`（接收消息）

### 第四步：连接 OpenClaw

```bash
# 添加飞书渠道（交互式引导）
openclaw channels add
# 选择 Feishu → 粘贴 App ID → 粘贴 App Secret

# 重启网关
openclaw gateway restart
```

### 第五步：测试 & 开机自启

在飞书中搜索你的机器人，发送消息测试。确认正常后设置开机自启：
```bash
openclaw gateway install
```

## 实用建议

- **长连接模式是首选**：不需要公网 IP、不需要域名、不需要云服务器，个人电脑或 NAS 即可运行
- **配对授权**：首次用户会收到配对码，管理员需在终端批准——这是安全特性，不是故障
- **群聊策略**：建议设置为"@机器人时才回复"，避免在群聊中过于活跃
- **Lark 国际版**：如果使用的是 Lark 国际版而非飞书中国版，需要用 Webhook + Cloudflare Tunnel 模式，教程中有详细说明

## 进阶：文档自动化与权限管理

> **如果你使用的是飞书官方插件**：文档、多维表格、日历、任务等操作已内置，不需要下面的额外安装步骤。

基础的飞书 AI 助手接入后（内置插件用户），你可以进一步安装飞书文档相关技能，实现"会后一句话整理纪要到飞书"等进阶场景。

### 安装飞书文档技能

```bash
npx playbooks add skill openclaw/openclaw --skill feishu-doc
npx playbooks add skill openclaw/openclaw --skill feishu-drive
npx playbooks add skill openclaw/openclaw --skill feishu-perm
```

| 技能 | 能力 |
|------|------|
| feishu-doc | 读写/创建/追加飞书文档 |
| feishu-drive | 飞书云盘文件管理 |
| feishu-perm | 文档/文件夹协作者权限管理 |

### 使用示例

会议结束后发一句话：

```text
把今天下午产品评审会的讨论整理成飞书文档，放到"会议纪要"文件夹里。
格式：结论 → 行动项（负责人 + 截止日期）→ 待确认事项。
把文档分享给参会的同事。
```

### 注意事项

- **必须先共享文件夹给 Bot**：飞书机器人没有"我的空间"根目录，如果不把目标文件夹分享给 Bot，创建/写入操作会失败
- **Wiki 也需要授权**：如果要写入知识库，需要先把 Bot 加入对应的 Wiki Space
- **权限管理谨慎使用**：feishu-perm 涉及权限变更，建议只给 Bot 分享特定文件夹，不给全盘权限

## 相关链接

- [feishu-openclaw 完整配置指南与社区支持](https://github.com/AlexAnys/openclaw-feishu) — 含官方 vs 内置插件对比、迁移指南、常见问题排查
- [飞书官方插件安装指南](https://github.com/AlexAnys/openclaw-feishu/blob/main/docs/feishu-official-plugin.md)
- [飞书官方图文教程](https://www.feishu.cn/content/article/7613711414611463386) — 飞书团队出品
- [OpenClaw 官方飞书文档](https://docs.openclaw.ai/zh-CN/channels/feishu)
- [腾讯云 - 保姆级教程：OpenClaw 接入飞书](https://cloud.tencent.com/developer/article/2626160)
- [腾讯云 - 5 分钟玩转飞书全家桶](https://cloud.tencent.com/developer/article/2631667)
