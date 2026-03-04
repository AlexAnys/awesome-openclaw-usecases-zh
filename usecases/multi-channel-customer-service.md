# 多渠道 AI 客户服务平台

> 含国内适配：企业微信 / 抖音 / 小红书

小型企业需要在多个应用间同时处理 WhatsApp、Instagram 私信、电子邮件和 Google 评价。客户期望全天候即时响应，但雇佣员工进行 24/7 覆盖成本高昂。

本用例将所有客户触点整合到一个由 AI 驱动的统一收件箱中，代表你进行智能回复。

## 功能概述

- **统一收件箱**：WhatsApp Business、Instagram 私信、Gmail 和 Google 评价集于一处
- **AI 自动回复**：自动处理常见问题、预约请求和一般咨询
- **人工交接**：将复杂问题升级或标记以供审核
- **测试模式**：在不影响真实客户的情况下向客户演示系统
- **业务上下文**：基于你的服务内容、定价和政策进行训练

## 真实商业案例

在 Futurist Systems，我们为本地服务企业（餐厅、诊所、美容院）部署此方案。一家餐厅将响应时间从 4 小时以上缩短到 2 分钟以内，80% 的咨询实现自动处理。

## 所需技能

- WhatsApp Business API 集成
- Instagram Graph API（通过 Meta Business）
- `gog` CLI 用于 Gmail
- Google Business Profile API 用于评价管理
- 在 AGENTS.md 中配置消息路由逻辑

## 如何设置

1. **通过 OpenClaw 配置连接各渠道**：
   - WhatsApp Business API（通过 360dialog 或官方 API）
   - Instagram（通过 Meta Business Suite）
   - Gmail（通过 `gog` OAuth）
   - Google Business Profile API 令牌

2. **创建业务知识库**：
   - 服务内容和定价
   - 营业时间和地址
   - 常见问题回复
   - 升级触发条件（如投诉、退款请求）

3. **在 AGENTS.md 中配置路由逻辑**：

```text
## 客户服务模式

当收到客户消息时：

1. 识别渠道（WhatsApp/Instagram/Email/Review）
2. 检查该客户是否启用了测试模式
3. 分类意图：
   - 常见问题 → 从知识库回复
   - 预约 → 检查可用时间，确认预订
   - 投诉 → 标记人工审核，确认收到
   - 评价 → 感谢反馈，回应关切

回复风格：
- 友好、专业、简洁
- 匹配客户的语言（ES/EN/UA）
- 绝不编造知识库中没有的信息
- 以商家名称结尾签名

测试模式：
- 回复前缀加 [TEST]
- 记录日志但不发送到真实渠道
```

4. **设置心跳检测（heartbeat）** 用于响应监控：

```text
## 心跳检测：客户服务检查

每 30 分钟：
- 检查超过 5 分钟未回复的消息
- 如果响应队列积压则发出警报
- 记录每日响应指标
```

## 关键洞察

- **语言检测很重要**：自动检测并使用客户的语言回复
- **测试模式必不可少**：客户需要在上线前看到系统运行效果
- **交接规则**：定义清晰的升级触发条件，避免 AI 越权
- **回复模板**：为敏感话题（退款、投诉）预先审批模板

## 相关链接

- [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp)
- [Instagram Messaging API](https://developers.facebook.com/docs/instagram-api/guides/messaging)
- [Google Business Profile API](https://developers.google.com/my-business)

## 中国用户适配

国内企业的客户触达渠道与海外完全不同。以下是针对国内主流渠道的适配方案。

### 渠道对照表

| 原版渠道 | 国内替代 | 可行性 |
|---------|---------|--------|
| WhatsApp Business | 企业微信客服 | 已验证可行 |
| Instagram DM | 抖音企业号私信 | 理论可行，需企业资质 |
| Gmail | 企业邮箱 | 直接可用，无需适配 |
| Google 评价 | 美团/大众点评 | 暂不可行（API 未开放） |

### P0 核心渠道：企业微信客服 API（已验证可行）

企业微信开放平台提供完整的客服消息 API，支持消息接收、主动回复和被动回复。本案例库已有详细的企业微信接入教程：

**前置条件**：参考 [企业微信 AI 助手](cn-wecom-ai-assistant.md) 完成基础接入。

**配置要点**：

1. 在企业微信管理后台创建自建应用，获取 Token 和 EncodingAESKey
2. 配置回调 URL 指向 OpenClaw 服务地址
3. 支持文本、图片、视频、文件等多种消息类型
4. 风险等级：低，使用官方 API

### P1 扩展渠道：抖音企业号私信（理论可行，需验证）

抖音开放平台提供企业号私信自动回复接口，可用于客服场景。

**关键限制**：

- 仅支持企业号，个人号不支持
- 回复私信不能发送链接
- 客户最后沟通后 48 小时内才能发消息
- 欢迎语每小时最多触发一次

### P1 扩展渠道：小红书私信通（理论可行，需验证）

小红书官方提供一站式线索经营平台[私信通](https://sxt.xiaohongshu.com/)。

**关键限制**：

- 需已认证的企业号 + 聚光平台认证
- 蓝 V 每日限 20 个陌生人主动消息，每人最多 3 条

### 暂不可行：美团/大众点评评价回复

美团开放平台支持查询评价，但评价自动回复 API 未公开。不满足"真实跑通"准入门槛，暂不收录。

### 多渠道参考实现

- **[ChatGPT-On-CS](https://github.com/cs-lazy-tools/ChatGPT-On-CS)**（开源 AGPL-3.0）：已支持微信/拼多多/千牛/B 站/抖音/小红书/知乎等多平台客服自动回复，可作为 OpenClaw 集成的技术参考

### 安全提醒

- 企业微信 API 凭证（Token、EncodingAESKey）通过环境变量配置，不要硬编码
- 抖音/小红书回复内容需遵守平台社区规范，避免触发风控
- 自动回复建议设置人工兜底机制，复杂问题及时转人工

> **安全提示**：所有平台的 API 密钥和 Token 属于敏感信息，请通过环境变量或 `.env` 文件配置，确保 `.env` 已加入 `.gitignore`。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/multi-channel-customer-service.md)
