# AI 辅助合同审查与 NDA 分拣

> 含国内适配：中国合同法条款库 / 电子签章（e签宝、法大大） / 飞书审批流集成

> **免责声明**：本用例中的 AI 辅助工具仅用于合同审查的初筛与标记，**不构成法律意见**。所有输出结果必须经持牌律师审核后方可用于商业决策。AI 无法替代法律专业判断——它是加速器，不是决策者。

法务团队每周要处理几十份合同——NDA、MSA、供应商协议、DPA……逐条审查既耗时又容易遗漏关键条款。你希望在合同进入正式法律审查之前，先用 AI 完成一轮标准化初筛，把明显的风险标红、偏离条款标黄、合规条款标绿。

这个工作流将合同审查拆解为两条自动化路径：

- **逐条审查**：上传合同后，AI 逐条比对你的协商 playbook，输出红/黄/绿标记 + 修改建议
- **NDA 快速分拣**：收到 NDA 后自动分类为"标准审批 / 法务复核 / 全面审查"，附带风险等级和建议操作
- **合规检查**：自动核验 GDPR、CCPA 等监管要求的合规清单

## 所需技能

- [Legal Productivity Plugin](https://github.com/anthropics/knowledge-work-plugins/tree/main/legal) — Anthropic 官方法律插件（6 条 slash 命令）
- [claude-legal-skill](https://github.com/evolsb/claude-legal-skill) — 开源合同审查技能（CUAD 风险检测 + 市场基准）
- MCP 集成：Box / Egnyte（文档存储）、Slack / Teams（通知）、Jira（任务追踪）

## 核心命令一览

| 命令 | 功能 | 输出 |
|------|------|------|
| `/review-contract` | 逐条审查合同，对照 playbook 标记偏差 | 红/黄/绿标记 + 修改建议（redline） |
| `/triage-nda` | NDA 快速分拣与风险预筛 | 分类路由 + 风险等级 + 建议操作 |
| `/compliance-check` | 监管合规核验（GDPR / CCPA / HIPAA） | 合规清单（通过/未通过）+ 修补建议 |
| `/risk-assessment` | 任意文档的法律风险评估 | 风险严重度分级 + 升级标准 |
| `/vendor-check` | 查询供应商现有协议状态 | NDA/MSA/DPA 到期日、关键条款 |
| `/brief` | 法务简报（每日摘要 / 专题研究 / 事件快报） | 结构化简报文档 |

## 如何设置

### 1. 安装法律插件

```bash
# Anthropic 官方插件
claude plugins add knowledge-work-plugins/legal

# 或使用开源 skill（支持 Claude Code / Codex / Cursor 等 26+ 工具）
git clone https://github.com/evolsb/claude-legal-skill ~/.claude/skills/contract-review
```

### 2. 配置协商 Playbook

在项目的 `.claude/` 目录下创建 `legal.local.md`，定义你的标准立场、可接受范围和升级触发条件：

```markdown
# Legal Playbook Configuration

## Contract Review Positions

### Limitation of Liability（责任限制）
- Standard position: Mutual cap at 12 months of fees paid/payable
- Acceptable range: 6-24 months of fees
- Escalation trigger: Uncapped liability, consequential damages inclusion

### Indemnification（赔偿条款）
- Standard position: Mutual indemnification for IP infringement and data breach
- Acceptable: Indemnification limited to third-party claims only
- Escalation trigger: Unilateral indemnification obligations, uncapped indemnification

### IP Ownership（知识产权归属）
- Standard position: Each party retains pre-existing IP; customer owns customer data
- Escalation trigger: Broad IP assignment clauses, work-for-hire provisions for pre-existing IP

### Data Protection（数据保护）
- Standard position: Require DPA for any personal data processing
- Requirements: Sub-processor notification, data deletion on termination, breach notification within 72 hours
- Escalation trigger: No DPA offered, cross-border transfer without safeguards

### Term and Termination（期限与终止）
- Standard position: Annual term with 30-day termination for convenience
- Acceptable: Multi-year with termination for convenience after initial term
- Escalation trigger: Auto-renewal without notice period, no termination for convenience

### Governing Law（管辖法律）
- Preferred: Delaware / 北京
- Acceptable: Major commercial jurisdictions (NY, CA, England & Wales)
- Escalation trigger: Non-standard jurisdictions, mandatory arbitration in unfavorable venue

## NDA Defaults
- Mutual obligations required
- Term: 2-3 years standard, 5 years for trade secrets
- Standard carveouts: independently developed, publicly available, rightfully received from third party
- Residuals clause: acceptable if narrowly scoped
```

### 3. 连接 MCP 工具（可选）

在 `.mcp.json` 中配置文档存储和通知集成：

```json
{
  "mcpServers": {
    "box": {
      "command": "npx",
      "args": ["@anthropic/mcp-server-box"],
      "env": {
        "BOX_CLIENT_ID": "${BOX_CLIENT_ID}",
        "BOX_CLIENT_SECRET": "${BOX_CLIENT_SECRET}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

> 凭证通过环境变量传递，不要硬编码在配置文件中。

### 4. 日常使用

**合同逐条审查：**

```text
/review-contract
# 上传合同文件或粘贴文本
# AI 会询问上下文：你是哪方？截止日期？重点关注领域？
# 然后逐条比对 playbook，输出标记报告
```

**NDA 分拣：**

```text
/triage-nda
# 上传 NDA 文件
# 输出：NDA 类型分类 → 期限/范围评估 → 风险等级 → 路由建议
```

## 红/黄/绿标记体系

审查结果按三级标记分类，每个条款获得一个颜色标记：

| 标记 | 含义 | 行动 |
|------|------|------|
| **🟢 绿色** | 条款符合 playbook 标准立场 | 无需行动 |
| **🟡 黄色** | 条款偏离首选条款，但在可接受范围内 | 建议审查，考虑协商 |
| **🔴 红色** | 条款存在重大风险，触发升级条件 | 必须协商或升级至高级法务 |

**红色触发示例**（基于 CUAD 风险分类）：

- 无上限赔偿条款（uncapped indemnification）
- 单方面修改权（unilateral amendment rights）
- 永久义务条款（perpetual obligations）
- 责任上限低于 6 个月费用
- 竞业限制超过 5 年
- 自动续约通知期少于 60 天

## NDA 分拣路由

| 路由 | 条件 | 处理方式 |
|------|------|---------|
| **标准审批** | 双向义务、标准期限、常规例外条款 | 自动放行或初级法务审批 |
| **法务复核** | 单向条款、非标准期限、范围偏宽 | 发送至法务团队人工复核 |
| **全面审查** | 无限期 NDA、竞业条款、跨境数据传输 | 高级法务全面审查 + 谈判 |

## 中国用户适配

国内合同审查有其独特的法律体系和工具生态。以下是针对中国法务团队的适配方案。

### 中美合同审查机制差异

| 维度 | 美国 | 中国 |
|------|------|------|
| 法律基础 | 普通法 + 州法差异 | 《民法典》合同编统一适用 |
| 合同自由度 | 高度自由，条款灵活 | 格式条款受限，强制性规定较多 |
| 电子签约 | DocuSign / Adobe Sign | e签宝 / 法大大 / 上上签 |
| 争议解决 | 诉讼/仲裁，管辖灵活 | 约定管辖有限制（与合同有实际联系的地点） |
| 数据合规 | CCPA / HIPAA（各州不同） | 《个人信息保护法》《数据安全法》（全国统一） |

### Playbook 条款库适配

将 playbook 的标准立场调整为中国法律框架：

```markdown
## 中国法合同审查标准

### 违约责任
- 标准立场：违约金不超过实际损失的 30%（参照最高院司法解释）
- 可接受范围：约定损失赔偿计算方式
- 升级触发：违约金明显过高（可能被法院调整）、排除主要义务的免责条款

### 知识产权
- 标准立场：委托开发成果归委托方，合作开发成果共有
- 升级触发：职务发明归属不明确、开源许可证冲突

### 保密条款
- 标准立场：保密期限 2-3 年，商业秘密不受期限限制
- 升级触发：竞业限制未约定补偿金（劳动合同法要求必须支付）

### 争议解决
- 首选：北京/上海仲裁委员会
- 可接受：合同履行地或被告住所地法院管辖
- 升级触发：境外仲裁条款（对国内企业不利）

### 数据合规（必查项）
- 必须：个人信息处理需取得同意或符合合法事由
- 必须：跨境数据传输需通过安全评估 / 标准合同 / 认证
- 升级触发：无数据处理协议、境外数据存储未经评估
```

### 电子签章集成

| 平台 | 说明 | 接入方式 |
|------|------|---------|
| **e签宝** | 市场份额领先，支持 CA 证书签章 | REST API，支持合同发起/签署/归档全流程 |
| **法大大** | 司法链存证，法院直接采信 | REST API + SDK，支持模板签署 |
| **上上签** | 企业级电子签约 | API 接入 |

可通过自定义 MCP server 对接电子签章平台：

```json
{
  "mcpServers": {
    "esign": {
      "command": "node",
      "args": ["mcp-server-esign/index.js"],
      "env": {
        "ESIGN_APP_ID": "${ESIGN_APP_ID}",
        "ESIGN_APP_SECRET": "${ESIGN_APP_SECRET}"
      }
    }
  }
}
```

### 飞书审批流集成

将合同审查结果对接飞书审批流程，实现"AI 初筛 → 人工复核 → 电子签署"全链路：

```text
向 OpenClaw 发送以下提示词：

当我上传合同文件时：
1. 使用 /review-contract 逐条审查，对照中国法 playbook
2. 生成审查报告（红/黄/绿标记）
3. 如果存在红色标记：自动在飞书创建审批单，抄送法务总监
4. 如果仅有黄色和绿色标记：发送飞书消息给法务专员确认
5. 审批通过后，自动通过 e签宝 发起签署流程

收到 NDA 时：
1. 使用 /triage-nda 快速分拣
2. 标准审批类：直接发送飞书通知，附带一键审批按钮
3. 法务复核类：创建飞书审批单，指定复核人
4. 全面审查类：创建飞书任务，关联到法务项目看板
```

### 国内合规要点

- **格式条款限制**：《民法典》第 496-498 条对格式条款有特殊要求，AI 审查应重点标记可能被认定为无效的格式条款
- **电子签名效力**：需使用经认证的 CA 机构签发的数字证书，简单电子签名在争议中证明力较弱
- **数据本地化**：《数据安全法》和《个人信息保护法》对跨境数据传输有严格限制，涉外合同必须审查数据条款
- **区块链存证**：部分平台（如法大大）支持司法链存证，签署过程直接上链，纠纷时法院可直接调取

### 国内推送渠道适配

| 原版方案 | 国内替代 | 说明 |
|---------|---------|------|
| Slack | **飞书** | 支持审批流 + 富文本卡片 + 机器人 |
| Slack | **钉钉** | Webhook 机器人，适合轻量通知 |
| Slack | **企业微信** | 企业用户首选，支持审批流 |
| Box | **飞书云文档** | 合同存储 + 在线预览 + 权限管控 |
| Jira | **飞书项目** | 法务任务看板 + 合同审查进度追踪 |

### 相关链接

- [Anthropic Legal Plugin](https://github.com/anthropics/knowledge-work-plugins/tree/main/legal) — 官方法律生产力插件
- [claude-legal-skill](https://github.com/evolsb/claude-legal-skill) — 开源合同审查技能（CUAD 风险检测）
- [e签宝开放平台](https://open.esign.cn/) — 电子签章 API
- [法大大开发者中心](https://open.fadada.com/) — 电子合同 API + 司法链存证
- [飞书开放平台](https://open.feishu.cn/) — 审批流 / 机器人 / 云文档 API

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/contract-review.md)
