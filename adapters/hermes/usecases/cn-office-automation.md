# Hermes 适配：办公自动化套件

> 这是 [办公自动化套件](../../../usecases/cn-office-automation.md) 的 Hermes 执行差异说明。完整背景、痛点和原始提示词请先阅读原用例。

## 一句话适配结论

`native`。本地文件整理、周报草稿、邮件摘要等子场景可直接用 Hermes 的文件、终端、定时任务能力完成；涉及真实邮箱/日历账号时再进入 `partial`。

## 概念映射

| OpenClaw | Hermes | 备注 |
|---|---|---|
| 文件系统工具 | `read_file` / `write_file` / `search_files` / `terminal` | 本地整理先在测试目录 dry-run |
| 定时任务 | `cronjob` | 邮件摘要/周报可定时推送 |
| 邮件 Skill | Hermes email skill / IMAP CLI | 真实邮箱需要授权码或 OAuth |

## Hermes 下的执行差异

1. **文件整理**：优先让 Hermes 在临时目录中试跑，确认分类规则后再操作真实下载目录。
2. **邮件自动化**：不要把邮箱授权码粘贴进对话；使用本地 `.env` 或系统钥匙串。
3. **周报生成**：Hermes 可先基于本地文本/邮件导出生成草稿，再由用户确认发送。

## 最小验证

```text
在一个临时目录创建 5 个测试文件：PDF、图片、表格、Markdown、超过 30 天的 txt。
请先 dry-run 展示分类计划，再把它们分别移动到 文档/图片/数据/归档。
```

期望：Hermes 输出移动计划并只在确认后写入。

## 已知限制 / 风险

- `writes_local`：真实目录操作前必须先列出计划。
- 邮件/日历属于 `credential_heavy`，第一轮适配只建议做摘要和草稿，不自动发送。
