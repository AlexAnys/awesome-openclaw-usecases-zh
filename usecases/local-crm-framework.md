# 本地 CRM 框架：DenchClaw

> 含国内适配：企业微信客户管理 / 钉钉宜搭

搭建一个真正能配合 OpenClaw 使用的 CRM 是件痛苦的事。你需要接入数据库、构建 UI、配置浏览器自动化、连接消息平台，还要让智能体理解你的数据结构。大多数人走到一半就放弃了，最后剩下一个半成品的 Notion 集成。

[DenchClaw](https://github.com/DenchHQ/DenchClaw) 是一个开源框架（MIT 协议），能把 OpenClaw 变成一个完全本地运行的 CRM、销售自动化和生产力平台——一条命令安装，所有数据都在你自己的机器上。

## 痛点

OpenClaw 作为底层原语非常强大，但把它用于真实业务流程（线索追踪、外呼、销售管道管理）需要拼接十多个工具：数据库、UI、浏览器访问、消息集成、文件管理。每新增一个集成就意味着更多手动配置、更多凭证管理、更多容易出问题的环节。你想要的是 Cursor 级别的业务操作体验，而不是一堆 shell 脚本。

## 它能做什么

| 功能 | 说明 |
|------|------|
| **一键安装** | `npx denchclaw` 安装全部组件——DuckDB 数据库、Web UI、OpenClaw 配置文件、浏览器自动化、技能——然后在 `localhost:3100` 打开 |
| **自然语言 CRM** | 问"给我看员工超过 5 人的公司"，视图实时更新，无需手动设置过滤器 |
| **完整浏览器自动化** | 复制你的 Chrome 配置文件，智能体拥有你的登录状态——说"把我 HubSpot 里的东西全导进来"它就会登录、导出、导入 |
| **6 种视图** | 表格、看板（Kanban）、日历、时间线（甘特图）、画廊、列表——全部可通过 YAML 由智能体配置 |
| **应用构建器** | OpenClaw 可在工作区内构建独立 Web 应用（仪表板、工具），直接访问你的数据 |
| **文件系统优先** | 表格过滤器、视图、列开关、日历设置——一切都是文件，OpenClaw 可以像编辑代码一样修改 UI |
| **同时是编码智能体** | DenchClaw 就是用 DenchClaw 自己构建的。它也是 Mac 上的完整文件浏览器和代码编辑器 |

## 如何设置

### 1. 一键安装

需要 Node 22+。

```bash
npx denchclaw
```

### 2. 完成引导向导

DenchClaw 会创建一个名为 `dench` 的专用 OpenClaw 配置文件，并在端口 19001 启动网关。

### 3. 打开浏览器

访问 `localhost:3100`。在 Safari 中可以添加到 Dock 作为 PWA 使用。

### 4. 开始对话

创建 CRM 对象：

```text
Hey, create a "Leads" object with fields: Name, Email, Company, Status (New/Contacted/Qualified/Won/Lost), and Notes. Import this CSV of leads I downloaded from Apollo.
```

> 中文说明：创建一个"线索"对象，包含姓名、邮箱、公司、状态（新建/已联系/已验证/成交/流失）和备注字段，并导入从 Apollo 下载的 CSV 文件。

查询和切换视图：

```text
Show me all leads where Status is "Contacted" and sort by last updated. Switch to Kanban view grouped by Status.
```

> 中文说明：显示所有状态为"已联系"的线索，按最后更新时间排序，切换到按状态分组的看板视图。

从 LinkedIn 导入线索：

```text
Go to my LinkedIn, find the last 20 people who viewed my profile, and add them as leads with their company info enriched.
```

> 中文说明：打开我的 LinkedIn，找到最近 20 个查看我主页的人，添加为线索并补充公司信息。

批量生成外呼邮件：

```text
Draft a personalized outreach email to each lead in "New" status based on their company's recent news. Save the drafts in a new "Outreach Drafts" document.
```

> 中文说明：为每个"新建"状态的线索，根据其公司最新动态撰写个性化外呼邮件，保存到新建的"外呼草稿"文档。

### 5. 多平台访问

可以通过 Telegram 或其他已连接的消息平台使用，随时随地管理销售管道。

## 所需技能

DenchClaw 自动捆绑所有必需技能，无需额外安装：

- **CRM 技能** — 基于 DuckDB 的结构化数据管理，支持对象、字段、条目和多种视图类型
- **应用构建器技能** — 在工作区内构建 Web 应用（仪表板、工具），可访问你的数据库
- **浏览器自动化技能** — Chromium 浏览器，使用你现有的 Chrome 登录状态，用于网页抓取、数据导入和外呼

## 关键洞察

- **文件系统 = 智能体原生 UI**：因为每个设置、过滤器和视图都存储为 YAML/Markdown 文件，OpenClaw 可以像编辑代码一样自然地修改 UI，不需要 API 封装层
- **DuckDB 是最佳平衡点**：最小、最高性能的嵌入式数据库，同时支持完整 SQL。无服务进程、无凭证、无网络——只是一个文件
- **Chrome 配置文件克隆是超能力**：不用和 OAuth 流程和 API 频率限制较劲，DenchClaw 直接复制你的浏览器登录状态。智能体看到你看到的，做你能做的
- **一条 `npx` 命令胜过一个周末的折腾**：整个技术栈（数据库、Web UI、OpenClaw 配置、网关、浏览器、技能）自动安装和配置。没有 Docker，没有 env 文件，没有依赖地狱

## 与 personal-crm 用例的区别

本仓库已有 [个人 CRM](personal-crm.md) 用例，两者定位不同：

| | 个人 CRM | DenchClaw 本地 CRM 框架 |
|---|---|---|
| **定位** | 个人联系人管理 | 小团队业务 CRM + 销售自动化 |
| **数据来源** | 邮件和日历自动发现 | 手动导入 + 浏览器自动化抓取 |
| **视图** | 数据库查询 | 6 种可视化视图（表格/看板/日历等） |
| **UI** | 无独立 UI | Web UI（localhost:3100） |
| **适合谁** | 个人职场社交管理 | 初创团队、自由职业者、小企业主的业务管道管理 |

## 演示

[观看演示视频](https://www.youtube.com/watch?v=pfACTbc3Bh4#t=43) — 展示从安装到用自然语言管理完整 CRM 管道的全流程。

## 中国用户适配

DenchClaw 本身是完全本地运行的工具，不依赖特定地区的云服务，因此国内用户可以直接使用。以下适配内容聚焦于如何将 DenchClaw 与国内业务生态结合，使其更好地服务中国小团队和个人创业者。

### 典型用户场景

| 用户画像 | 痛点 | DenchClaw 怎么帮 |
|---------|------|-----------------|
| **初创公司 BD** | 用 Excel 管客户、跟进靠记忆、数据散落在微信/邮件/飞书 | 统一客户管道 + 自然语言查询 + 看板视图追踪阶段 |
| **自由职业者/顾问** | 客户少但关系深，需要记住每次沟通细节 | 时间线视图 + 备注管理 + 跟进提醒 |
| **电商小店主** | 供应商和分销商信息杂乱，比价和补货靠人工 | 结构化供应商数据库 + 浏览器自动化抓取价格 |
| **微商/社群运营** | 客户分层管理困难，群发容易被封号 | 客户标签分组 + 个性化内容生成（不直接操作微信） |

### 企业微信客户数据集成

企业微信提供完整的[客户联系 API](https://developer.work.weixin.qq.com/document/path/92114)，可以将外部联系人数据导入 DenchClaw 进行统一管理。

**集成思路**：

1. 通过企业微信 API 获取外部联系人列表和详情
2. 编写脚本将数据导出为 CSV
3. 导入 DenchClaw 进行统一管理和分析

**示例脚本**（导出企业微信客户到 CSV）：

```bash
# 获取 access_token
export WECOM_CORP_ID="你的企业ID"
export WECOM_SECRET="你的应用Secret"

# 获取 token（实际使用时请封装为脚本）
TOKEN=$(curl -s "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=${WECOM_CORP_ID}&corpsecret=${WECOM_SECRET}" | jq -r '.access_token')

# 获取外部联系人列表
curl -s "https://qyapi.weixin.qq.com/cgi-bin/externalcontact/list?access_token=${TOKEN}&userid=你的员工ID" | jq '.external_userid[]'
```

获取到客户列表后，可以进一步调用获取客户详情接口，整理成 CSV 后导入 DenchClaw：

```text
Import this CSV of WeChat Work contacts. Create fields: Name, Company, Tags, Last Contact Date, Follow-up Status, and Notes.
```

> 中文说明：导入这份企业微信联系人 CSV。创建字段：姓名、公司、标签、最后联系日期、跟进状态和备注。

**企业微信客户标签同步**：

企业微信支持[企业客户标签管理 API](https://developer.work.weixin.qq.com/document/path/96320)，最多可配置 10000 个标签。可以定期同步标签到 DenchClaw，保持数据一致：

```text
Update the Tags field for all contacts based on this tag mapping CSV exported from WeChat Work. Show me contacts where Tags contain "高意向" in Kanban view grouped by Follow-up Status.
```

> 中文说明：根据从企业微信导出的标签映射 CSV 更新所有联系人的标签字段。用看板视图显示标签包含"高意向"的联系人，按跟进状态分组。

### 钉钉宜搭 CRM 数据迁移

如果团队已在使用钉钉宜搭搭建的低代码 CRM，可以将数据导出后迁移到 DenchClaw，获得更灵活的自然语言查询和自动化能力。

**迁移步骤**：

1. 在宜搭管理后台 → 数据管理 → 选择表单 → 导出数据，将现有 CRM 数据导出为 Excel/CSV
2. 在 DenchClaw 中创建对应的对象和字段
3. 导入数据

```text
Create a "客户" object with fields matching this CSV header: 客户名称, 联系人, 电话, 行业, 签约金额, 跟进阶段, 最后跟进时间. Import the data and switch to Kanban view grouped by 跟进阶段.
```

> 中文说明：创建一个"客户"对象，字段匹配 CSV 表头，导入数据并切换到按跟进阶段分组的看板视图。

### 常用业务查询 Prompt

DenchClaw 支持自然语言查询，以下是常用的中文业务查询示例：

**客户查询类**：

```text
Show me all contacts where 跟进阶段 is "需求确认" and 最后跟进时间 is more than 7 days ago. Sort by 签约金额 descending.
```

> 查找所有处于"需求确认"阶段且超过 7 天未跟进的客户，按签约金额降序排列。

**管道分析类**：

```text
Give me a summary: how many leads are in each stage? What's the total potential revenue in "报价中" stage? Which deals haven't been updated in 2 weeks?
```

> 生成管道摘要：每个阶段有多少线索？"报价中"阶段的潜在总金额是多少？哪些交易超过两周没有更新？

**批量操作类**：

```text
For all contacts tagged "展会-2026深圳", draft a follow-up email thanking them for visiting our booth and suggesting a demo call next week. Save drafts in a "展会跟进" document.
```

> 为所有标记为"展会-2026深圳"的联系人，撰写感谢参观展位的跟进邮件并建议下周安排演示通话，保存到"展会跟进"文档。

### 浏览器自动化在国内的应用

DenchClaw 的浏览器自动化功能（克隆 Chrome 配置文件）在国内场景同样适用：

- **从 1688/阿里巴巴导入供应商信息**：自动抓取供应商页面，提取公司名、主营产品、联系方式。阿里系反爬机制严格，注意控制请求频率
- **从企查查/天眼查补充公司信息**：自动查询客户公司的工商信息、融资情况。商业数据平台，频繁请求会被封号，仅建议小量手动辅助查询

> **注意**：浏览器自动化相当于用你的账号自动操作，请遵守各平台的使用条款，控制操作频率，避免触发风控。

### 风险提示

- **项目早期阶段**：DenchClaw 于 2026 年 2 月上线，目前处于快速迭代期，GitHub 已获得社区广泛关注。生产环境使用建议关注版本更新
- **浏览器自动化权限**：Chrome 配置文件克隆意味着智能体拥有你的全部登录状态，务必了解其访问范围
- **数据安全**：所有数据存储在本地 DuckDB 文件中，不上传云端。但请做好本地备份
- **Node.js 版本**：需要 Node 22+，部分国内镜像源可能更新不及时，建议使用 `nvm` 管理版本
- **网络环境**：`npx denchclaw` 安装时需要访问 npm registry，国内用户如遇网络问题可配置镜像源：
  ```bash
  npm config set registry https://registry.npmmirror.com
  npx denchclaw
  ```

### 和已有用例的配合

- 配合 [企业微信 AI 助手](cn-wecom-ai-assistant.md)：将企业微信作为 DenchClaw 的消息入口，在企业微信中查询和更新客户信息
- 配合 [多渠道 AI 客户服务](multi-channel-customer-service.md)：DenchClaw 管理客户数据，客服系统处理客户消息，两者互补
- 配合 [AI 财报追踪器](earnings-tracker.md)：追踪客户公司的财报动态，辅助 BD 决策

## 相关链接

- [DenchClaw GitHub](https://github.com/DenchHQ/DenchClaw) — MIT 开源协议
- [DenchClaw 官网](https://denchclaw.com)
- [Discord 社区](https://discord.gg/PDFXNVQj9n)
- [技能商店](https://skills.sh)
- [Hacker News 讨论](https://news.ycombinator.com/item?id=44228973) — 社区反馈
- [企业微信客户联系 API](https://developer.work.weixin.qq.com/document/path/92114) — 外部联系人管理接口
- [企业微信客户标签 API](https://developer.work.weixin.qq.com/document/path/96320) — 企业客户标签管理
- [钉钉宜搭低代码 CRM](https://www.aliwork.com/) — 钉钉低代码平台

---
**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/62)
