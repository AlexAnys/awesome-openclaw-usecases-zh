# AI 文件管理器（Claw Drive）

> 含国内适配：坚果云 WebDAV 同步 / 本地化分类 / 中文搜索场景

不再手动整理文件。把文件发给你的 OpenClaw 智能体，它会自动分类、打标签、去重并建立索引。之后用自然语言就能找到任何文件——不用再翻无尽的文件夹。

## 痛点

- 文件散落在 `新建文件夹`、`新建文件夹(2)`、`最终版`、`最终版-真的最终版` 里
- 找一张保险卡或体检报告要花 10 分钟翻文件夹
- 到处都是重复下载的文件
- 没有一致的命名和组织方式

## 它能做什么

| 功能 | 说明 |
|------|------|
| **自动分类** | 发送文件（本地、邮件、Telegram）→ 智能体自动放入合适的目录（`documents/`、`finance/`、`medical/` 等） |
| **自动打标签** | 根据内容或文件名生成描述和标签 |
| **去重** | SHA-256 哈希校验，防止同一文件存两份 |
| **隐私优先** | 默认启用敏感模式——如果你没有明确允许读取内容，它只使用文件名 |
| **自然语言搜索** | "我猫的疫苗记录在哪？" → 立即找到结果 |
| **云端备份** | 可选的 Google Drive 同步（通过 fswatch + rclone） |

## 效果示例

你说："保存这张车险保单"

```
Stored: insurance/car-insurance-id-2026.pdf
   Tags: insurance, auto, honda-civic
   Description: Acme Insurance, policy ****3441
```

三个月后你问："我的车险保单号是多少？" → 立即回答。

## 所需技能

[Claw Drive](https://github.com/dissaozw/claw-drive) — 开源 CLI + JSONL 索引，可作为 OpenClaw 技能使用。

## 如何设置

### 1. 克隆并安装技能

```bash
git clone https://github.com/dissaozw/claw-drive.git ~/.openclaw/skills/claw-drive
cd ~/.openclaw/skills/claw-drive && bash install.sh
```

也可通过 Homebrew 安装：

```bash
brew install dissaozw/tap/claw-drive
claw-drive init
```

### 2. 告诉智能体如何使用

将以下提示词发送给你的 OpenClaw 智能体：

> 中文说明：告诉智能体用 Claw Drive 管理文件，对敏感文件启用隐私模式。

```text
I want you to manage my files using Claw Drive. When I send you any file,
store it with appropriate category and tags. For sensitive documents (medical,
financial, legal), use sensitive mode — don't read the content, just use the
filename and any description I give you. For everything else, you can read
the content to generate better tags.
```

### 3. 可选——设置云端备份

> 中文说明：设置 Google Drive 自动同步备份。

```text
Set up Google Drive sync for my Claw Drive files so everything is backed up
automatically.
```

## 相关链接

- [Claw Drive 仓库](https://github.com/dissaozw/claw-drive)
- [OpenClaw 官方文档](https://docs.openclaw.ai)

---

## 中国用户适配

国内用户的文件管理场景和云存储生态与海外有明显差异。以下是针对国内环境的适配方案。

### 国内文件管理痛点

国内用户除了通用的"文件乱"问题外，还有一些特有的场景：

- **发票满天飞**：增值税发票、电子发票 PDF 散落在微信、邮箱、网页下载里，报销时找不到
- **证件管理**：身份证、户口本、房产证、学历证的扫描件需要安全存储和快速查找
- **合同归档**：租房合同、劳动合同、保险合同等需要按到期日追踪
- **素材库管理**：自媒体运营者的图片、视频素材散落在多个设备和平台

### 本地化分类目录建议

原版 Claw Drive 的默认分类偏向欧美场景。以下是推荐的中国用户分类目录：

```
claw-drive/
├── invoice/          # 发票（增值税发票、电子发票、定额发票）
├── contract/         # 合同（租房、劳动、保险、购房）
├── certificate/      # 证件（身份证、户口本、学历、资格证）
├── finance/          # 财务（银行流水、信用卡账单、理财记录）
├── medical/          # 医疗（体检报告、病历、处方、医保记录）
├── insurance/        # 保险（车险、医疗险、寿险保单）
├── property/         # 房产（房产证、购房合同、物业单据）
├── tax/              # 税务（个税申报、完税证明）
├── education/        # 教育（成绩单、毕业证、培训证书）
├── work/             # 工作（简历、offer、离职证明、社保记录）
├── media/            # 素材（图片、视频、音频素材）
└── general/          # 通用（其他未分类文件）
```

通过以下提示词让智能体使用中国化分类：

> 中文说明：配置智能体使用中国化的文件分类目录。

```text
For my Claw Drive file organization, use these China-specific categories:
- invoice/ for fapiao (发票), VAT invoices, e-invoices
- contract/ for rental, employment, insurance, and purchase contracts
- certificate/ for ID cards, hukou, diplomas, professional certificates
- finance/ for bank statements, credit card bills, investment records
- medical/ for health checkups, medical records, prescriptions
- insurance/ for auto, health, life insurance policies
- property/ for property certificates, purchase contracts, utility bills
- tax/ for individual income tax filings, tax clearance certificates
- education/ for transcripts, graduation certificates, training certificates
- work/ for resumes, offer letters, resignation certificates, social security records
- media/ for image, video, and audio assets

When categorizing Chinese documents, pay attention to:
- Files with "发票" or "invoice" in the name go to invoice/
- Files with "合同" or "contract" in the name go to contract/
- Track expiration dates for contracts and certificates
```

### 云端备份：坚果云 WebDAV（替代 Google Drive）

Google Drive 在国内无法直接使用。坚果云是国内最适合替代的方案，原因如下：

- 原生支持 WebDAV 协议，rclone 可直接对接
- 国内访问稳定，无需代理
- 免费版提供 1GB/月上传流量，个人文件管理足够
- 数据存储在国内服务器，合规性好

#### 配置步骤

**1. 获取坚果云 WebDAV 应用密码**

登录坚果云网页版 → 右上角账户信息 → 安全选项 → 第三方应用管理 → 添加应用 → 生成应用密码。

**2. 配置 rclone 连接坚果云**

```bash
rclone config
# 选择 n (new remote)
# 名称输入: jianguoyun
# 类型选择: webdav
# URL 输入: https://dav.jianguoyun.com/dav/
# 供应商选择: other
# 用户名: 你的坚果云注册邮箱
# 密码: 上一步生成的应用密码
```

**3. 验证连接**

```bash
rclone lsd jianguoyun:
```

**4. 让智能体使用坚果云同步**

> 中文说明：将 Google Drive 同步替换为坚果云 WebDAV 同步。

```text
Set up cloud sync for my Claw Drive files using Jianguoyun (坚果云) WebDAV
instead of Google Drive. The rclone remote name is "jianguoyun".
Sync to the path jianguoyun:claw-drive/.
Do NOT sync the identity/ and certificate/ folders — keep those local only.
```

#### 坚果云使用注意事项

- 免费版限制：每 30 分钟最多 600 次请求，单文件最大 500MB
- 避免短时间大量同步操作，否则会触发系统保护机制导致账号临时冻结
- 建议设置同步间隔为 5 分钟以上，而非实时同步
- 敏感文件（证件、身份信息）建议仅保留本地，不同步到云端

### 国内网盘方案对比

| 网盘 | WebDAV | rclone 直连 | 免费空间 | 推荐度 | 说明 |
|------|:------:|:-----------:|---------|:------:|------|
| **坚果云** | 原生支持 | 支持 | 1GB/月上传 | 推荐 | WebDAV 协议成熟，rclone 直连最简单 |
| 阿里云盘 | 不支持 | 需 AList 中转 | 100GB+ | 可选 | 空间大但需额外部署 AList 提供 WebDAV 接口 |
| 百度网盘 | 不支持 | 需 AList 中转 | 5GB | 不推荐 | 非会员限速严重，配置复杂 |
| 115 网盘 | 不支持 | 需 AList 中转 | — | 不推荐 | 需付费会员，API 不稳定 |

**推荐方案**：坚果云直连最省心。如果需要大容量存储，可以用 AList 作为中间层桥接阿里云盘，但配置复杂度较高，适合有一定技术基础的用户。

### 中文搜索场景示例

Claw Drive 的自然语言搜索在中文场景下特别实用：

| 你的问题 | 智能体会找到 |
|---------|------------|
| "去年的体检报告" | `medical/health-checkup-2025.pdf` |
| "和房东签的租房合同" | `contract/rental-agreement-2024.pdf` |
| "上个月的增值税发票" | `invoice/vat-invoice-202602.pdf` |
| "驾驶证照片" | `certificate/driving-license.jpg` |
| "平安车险保单号是多少" | 从 `insurance/pingan-auto-2026.pdf` 的索引中提取 |
| "所有快到期的合同" | 根据索引中的到期日期字段筛选 |

### 实用建议

- **从小处开始**：先用一个文件夹（比如 `~/Downloads`）测试分类效果，满意后再扩大范围
- **安全第一**：证件类文件（身份证、户口本）建议启用 sensitive mode 并禁止云同步
- **发票场景**：如果日常有大量发票需要管理，可以搭配微信小程序"票据助手"扫描后导入
- **批量迁移**：已有的文件可以用 Claw Drive 的 reindex 功能批量导入和分类，不需要逐个处理
- **中文文件名**：Claw Drive 基于 JSONL 索引，原始文件名会被保留，支持中文文件名搜索

> **安全提示**：证件扫描件属于高度敏感信息。请确保 OpenClaw 运行在安全环境中，启用 Claw Drive 的 sensitive mode，并且不要将 `identity/` 和 `certificate/` 目录同步到任何云端。

### 相关链接

- [坚果云 WebDAV 帮助文档](https://help.jianguoyun.com/?p=2064) — 第三方应用授权配置
- [rclone 官方文档](https://rclone.org/webdav/) — WebDAV 配置指南
- [AList 项目](https://github.com/AlistGo/alist) — 多网盘统一 WebDAV 接口（桥接阿里云盘等）
