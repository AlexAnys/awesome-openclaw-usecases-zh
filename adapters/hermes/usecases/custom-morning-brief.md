# Hermes 适配：自定义早间简报

> 这是 [自定义早间简报](../../../usecases/custom-morning-brief.md) 的 Hermes 执行差异说明。完整背景、痛点和原始提示词请先阅读原用例。

## 一句话适配结论

`adapter`。核心逻辑非常适合 Hermes：`cronjob` 负责定时，web/search 负责信息收集，Slack/Telegram/Discord/local 负责推送；飞书/钉钉需额外 adapter。

## 概念映射

| OpenClaw | Hermes | 备注 |
|---|---|---|
| `openclaw cron add` | `cronjob` | prompt 必须自包含，未来 cron run 没有当前聊天上下文 |
| Telegram/Discord/iMessage | Hermes delivery target | 可用 `origin` / `slack` / `telegram` / `local` |
| 飞书/钉钉推送 | Lark CLI / webhook / 后续 adapter | Hermes 当前不应假设原生飞书 channel |

## Hermes 下的执行差异

1. **定时任务 prompt 要自包含**：写清主题、信息源、输出格式、发送目标和时区。
2. **时区**：显式使用 `Asia/Shanghai`。
3. **推送目标**：如果没有飞书 adapter，先投递到当前 Slack 线程或 `local`。
4. **主动建议**：适合保留，但建议只列候选任务，不自动执行外部写操作。

## 最小验证

```text
创建一个一次性的早间简报测试任务：5 分钟后运行，内容包括 AI/创业/科技新闻、今天待办、你建议我可以自动化的 3 件事。先发送回当前聊天，不要发外部渠道。
```

期望：Hermes 创建 one-shot cron，并在目标时间返回结构化简报。

## 已知限制 / 风险

- `external_write`：发送到群聊、飞书、邮件前需要确认。
- 新闻源可能受网络影响，prompt 应要求“无法访问时说明缺口，不要编造”。
