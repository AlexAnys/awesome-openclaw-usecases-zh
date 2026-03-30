# 用 Lark CLI 让 Agent 操作飞书

在飞书里和 AI 聊天只是起点。更大的价值是：**让 AI 直接帮你操作飞书里的一切**——搜文档、读妙记、查日历、发消息、改多维表格，你只需要用自然语言说一句话。

飞书官方于 2026-03-28 开源了 [Lark CLI](https://github.com/larksuite/cli)（MIT），这是一个 CLI（命令行工具），覆盖飞书 11 个业务领域、200+ 命令，内置 19 个 AI Agent Skills（智能体技能包）。装好之后，你的 AI Agent 就能以你的身份操作飞书。

## 它能做什么

| 领域 | 能力 | 你说一句话 |
|------|------|-----------|
| 💬 消息 | 发消息、建群、搜群、搜消息 | "帮我搜一下项目组的群" |
| 📄 文档 | 创建/读写/搜索文档 | "搜一下飞书里关于 OKR 的文档" |
| 📊 多维表格 | 表/字段/记录/视图/仪表盘 | "帮我在多维表格里加一条记录" |
| 📈 电子表格 | 创建/读写/追加/导出 | "把这个表格的数据拉出来" |
| 📅 日历 | 日程查询、建活动、忙闲查询 | "帮我看看今天有什么日程" |
| 📧 邮箱 | 浏览/搜索/发/回复邮件 | "看看飞书邮箱最近有什么邮件" |
| ✅ 任务 | 创建/查询/完成任务 | "帮我建个飞书任务" |
| 📚 知识库 | 空间/节点管理 | "找一下知识库里的文档" |
| 👤 通讯录 | 按姓名/邮箱/手机搜人 | "飞书里搜一下张三的联系方式" |
| 🎥 会议/妙记 | 搜录制、查会议纪要/转写 | "找一下昨天的妙记，把内容拉出来" |
| 📁 云空间 | 上传下载文件、权限管理 | "帮我把这个文件传到飞书云空间" |

### 和"飞书 AI 助手"用例的区别

| | 飞书 AI 助手 | 本用例（Lark CLI） |
|---|---|---|
| **做什么** | 在飞书里和 AI 对话 | 让 AI 帮你操作飞书 |
| **方向** | 飞书 → AI → 飞书（聊天） | AI → 飞书（主动操作） |
| **身份** | 机器人身份 | 你本人的身份（OAuth 开放授权） |
| **覆盖范围** | 消息收发为主 | 11 个业务领域，200+ 命令 |
| **依赖** | 需要配置飞书插件 | 独立运行，不依赖 OpenClaw 飞书插件 |

**两者互补**：飞书 AI 助手让你在飞书里和 AI 对话，Lark CLI 让 AI 主动帮你操作飞书里的一切。可以同时使用。

## 所需技能

- [Lark CLI](https://github.com/larksuite/cli)（4,400+ stars，MIT，飞书官方维护）
- Node.js / npm（安装 CLI 和 Agent Skills 需要）
- 飞书账号（个人版或企业版均可）

> Lark CLI 自带 19 个 Agent Skills，安装时会自动 symlink（符号链接）到 OpenClaw / Claude Code / Cursor 等 Agent 框架，不需要额外安装 OpenClaw 技能。

## 如何设置

### 一句话安装

把下面这段话发给你的 AI Agent：

```text
帮我安装飞书 Lark CLI。
参考这个指南：https://github.com/AlexAnys/openclaw-feishu/blob/main/docs/lark-cli-guide.md
```

Agent 会自动完成安装和配置，然后给你一个授权链接。你在浏览器中打开、用飞书账号确认——就这一步。

> 不需要提前创建飞书应用，CLI 会帮你自动创建。

### 手动安装（如果你想自己来）

```bash
# 安装
npm install -g @larksuite/cli

# 创建飞书应用并配置（会给你一个浏览器链接，点击确认）
lark-cli config init --new

# 登录授权（也是一个浏览器链接，点击确认）
lark-cli auth login --domain all

# 安装 AI Agent Skills
npx skills add larksuite/cli -y -g

# 验证
lark-cli doctor
```

## 使用示例

装好后直接跟你的 AI 说就行。以下是实测可用的场景（2026-03-30 验证）：

### 搜索文档

```text
你：搜一下飞书里关于"周报"的文档
```

Agent 执行 `lark-cli docs +search --query "周报" --format pretty`，返回文档列表（标题、类型、修改时间、链接）。

### 阅读妙记智能纪要

```text
你：找一下昨天的妙记，把纪要内容拉出来
```

Agent 先搜索 `lark-cli docs +search --query "文字记录"`，找到妙记文档后用 `lark-cli docs +fetch --doc "文档URL"` 读取完整的智能纪要——包括总结、章节、关键决策、金句。

### 查日历

```text
你：帮我看看今天有什么日程
```

Agent 执行 `lark-cli calendar +agenda`，返回当天的所有日程。

### 搜群

```text
你：帮我搜一下飞书里叫"项目组"的群
```

Agent 执行 `lark-cli im +chat-search --query "项目组"`，返回群名、群 ID、创建时间等。

### 搜人

```text
你：飞书里搜一下张三
```

Agent 执行 `lark-cli contact +search-user --query "张三"`，返回姓名、部门、头像等。

## 实用建议

- **令牌有效期**：Access Token（访问令牌）2 小时（自动续期），Refresh Token（刷新令牌）7 天。超过 7 天没用需要重新授权一次——跟 Agent 说"重新授权飞书"就行
- **输出格式**：对 Agent 来说 `--format json`（默认）最好解析；对人来说 `--format pretty` 或 `--format table` 最直观
- **`--dry-run`（预演模式）**：在执行可能产生副作用的操作前（如发消息、改记录），可以用 `--dry-run` 先预览请求
- **`--page-all`**：查询结果可能分页，加上 `--page-all` 自动获取全部
- **和飞书 AI 助手共存**：Lark CLI 使用 OAuth 用户身份，和飞书插件的机器人身份互不干扰，可以同时使用

## 相关链接

- [Lark CLI GitHub](https://github.com/larksuite/cli) — 官方源码 & 完整文档
- [Lark CLI 上手指南](https://github.com/AlexAnys/openclaw-feishu/blob/main/docs/lark-cli-guide.md) — 安装、配置、Agent 技术参考
- [飞书开放平台](https://open.feishu.cn) — API 文档
- [openclaw-feishu](https://github.com/AlexAnys/openclaw-feishu) — 飞书 × OpenClaw 配置指南
