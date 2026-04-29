# Hermes 适配：中文互联网 30 天研究工具

> 这是 [中文互联网 30 天研究工具](../../../usecases/cn-internet-research-30days.md) 的 Hermes 执行差异说明。完整背景、痛点和原始提示词请先阅读原用例。

## 一句话适配结论

`adapter`。Hermes 适合作为“中文互联网研究员”：用 terminal 安装和运行 CLI，用 browser/Playwright 增强采集，用 subagent 做并行分析。但 8 平台不是零配置稳定覆盖，应按数据源可用性分层。

## 概念映射

| OpenClaw | Hermes | 备注 |
|---|---|---|
| Skill 安装 | 临时 clone / `~/.hermes/skills` / 外部 CLI | 第一轮建议临时 venv 验证 |
| 浏览器技能 | browser / Playwright | 微博、小红书、抖音依赖反爬情况 |
| 报告推送 | `send_message` / local file | 正式研究建议同时保存 Markdown 原文 |

## Hermes 下的执行差异

1. **安装隔离**：在 venv 中安装 `jieba`、可选 `playwright`，避免污染系统 Python。
2. **零配置能力**：先运行 `--diagnose`，再决定搜索哪些平台。
3. **结果可信度**：报告中必须标注哪些平台失败、哪些结果日期未知。
4. **深度研究**：可用 Hermes 子任务并行拆分平台和主题，但最终报告要合并去重。

## 最小验证

```bash
git clone https://github.com/Jesseovo/last30days-skill-cn.git /tmp/last30days-skill-cn
python -m venv /tmp/last30days-venv
/tmp/last30days-venv/bin/pip install jieba requests beautifulsoup4 lxml
/tmp/last30days-venv/bin/python /tmp/last30days-skill-cn/scripts/last30days.py --diagnose
/tmp/last30days-venv/bin/python /tmp/last30days-skill-cn/scripts/last30days.py "AI编程助手" --emit compact --quick
```

期望：至少返回部分 B站/百度/微信或其他来源；失败平台被明确列出。

## 已知限制 / 风险

- `read_only`，但涉及爬虫合规。不要高频、大规模采集。
- 小红书/微博/抖音可能 timeout 或被反爬；适配文档不能承诺 8 平台稳定。
- 微信公众号没有 Key 时可能只能通过搜狗等间接来源。
