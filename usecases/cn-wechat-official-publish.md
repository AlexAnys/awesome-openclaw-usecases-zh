# 微信公众号 Markdown 自动发布

写完一篇 Markdown 文章后，手动复制到公众号后台、反复调格式、再扫码登录发布，流程很长且容易中断。尤其在飞书/Telegram 里协作写作时，切来切去会明显降低效率。

这个用例把 OpenClaw 作为发布入口：你在对话里说一句自然语言（如"帮我把这篇文章发到微信公众号上去"），智能体就会调用 `publish_wechat` Skill，把原始 Markdown 交给 Gateway 创建发布任务，并按状态引导你完成扫码登录与确认。

## 它能做什么

- **对话式发起发布**：在飞书/Telegram/Web 对话中直接触发公众号发布任务
- **原始 Markdown 直传**：OpenClaw 不做 `markdown -> html` 转换，内容处理全部在 Gateway
- **登录态闭环**：未登录时返回二维码图片，扫码后可继续 `confirm`
- **状态可追踪**：随时查询任务状态，成功/失败都会返回 `task_id` 和 `status`
- **错误可读**：对常见错误码（如 `409/422/502`）做友好提示，同时保留原始 `error.code`

## 所需技能

- 自定义 Skill：`publish_wechat`
  - 支持命令：
    - `/publish_wechat`
    - `/publish_wechat status <task_id>`
    - `/publish_wechat confirm <task_id>`
    - `/publish_wechat relogin`
- 后端服务：`openclaw-wechat-gateway`（负责与发布 Agent 交互）

## 如何设置

1. 部署并启动 Gateway 服务，确认可访问发布接口。
2. 在 OpenClaw 安装 `publish_wechat` Skill（确保运行脚本可调用 Gateway）。
3. 在对话里粘贴文章 Markdown，发送自然语言：

```text
帮我把这篇文章发到微信公众号上去
```

4. 如返回 `waiting_login`，OpenClaw 会发送二维码图片，使用微信扫码登录。
5. 登录后发送：

```text
/publish_wechat confirm <task_id>
```

6. 需要查看进度时发送：

```text
/publish_wechat status <task_id>
```

7. 如登录态异常，可先重置登录态再重新发起发布：

```text
/publish_wechat relogin
```

## 可复制提示词

```text
把这篇文章发到微信公众号，标题就用第一行 H1，不要改正文内容。
```

```text
登录成功了，继续发布刚才那个任务。
```

## 实用建议

- **正文前先自检图片**：若文章里有外链图，先确认可公开访问，避免发布阶段触发图片策略失败。
- **等待登录时不要重复发 publish**：先扫码，再 `confirm`，避免创建重复任务。
- **给每篇文章唯一标题**：便于在状态查询和日志里快速定位任务。

## 风控与合规提醒

> ⚠️ 涉及公众号自动化时，请遵守微信公众平台规范与账号安全策略。建议先用测试号/低风险账号验证流程，再用于正式账号。不要批量高频发布，不要绕过平台风控机制。

## 相关链接

- [openclaw-wechat-gateway（实现仓库）](https://github.com/xu75/openclaw-wechat-gateway)
- [OpenClaw 官方仓库](https://github.com/openclaw/openclaw)
