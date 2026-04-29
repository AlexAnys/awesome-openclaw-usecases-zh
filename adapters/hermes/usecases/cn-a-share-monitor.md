# Hermes 适配：A 股每日行情监控

> 这是 [A 股每日行情监控](../../../usecases/cn-a-share-monitor.md) 的 Hermes 执行差异说明。完整背景、痛点和原始提示词请先阅读原用例。

## 一句话适配结论

`partial`。Hermes 很适合做定时信息整理和健康检查，但不应做交易决策。AKShare 在不同网络环境下可用性差异明显，必须内置 fallback 和失败说明。

## 概念映射

| OpenClaw | Hermes | 备注 |
|---|---|---|
| Python 沙箱 | venv + `terminal` | 安装 `akshare` 后生成 Markdown 报告 |
| MCP 服务 | Hermes MCP / terminal 调用 | MCP 公共服务可能变更，生产建议自建 |
| 飞书推送 | `send_message` / Lark CLI | Slack/local 可作为默认验证目标 |
| 定时盘前/盘后 | `cronjob` | prompt 必须写明非投资建议 |

## Hermes 下的执行差异

1. **数据源健康检查优先**：先测试 AKShare 接口，再生成报告。
2. **网络标签**：标为 `cn_preferred`，海外/代理环境下东方财富接口可能失败。
3. **输出定位**：只做信息整理、异常提醒、数据源状态说明，不给买卖建议。
4. **定时任务**：每次运行都要附“接口健康检查”和数据时间。

## 最小验证

```bash
python -m venv /tmp/a-share-venv
/tmp/a-share-venv/bin/pip install akshare pandas
/tmp/a-share-venv/bin/python - <<"PY"
import akshare as ak
print(ak.stock_market_activity_legu().head().to_string())
PY
```

期望：能输出市场活跃度；如东方财富接口失败，应把失败写入报告而不是中断。

## 已知限制 / 风险

- `financial`：不是投资建议，不自动交易。
- 数据接口依赖公开网站，可能限速或地区不可用。
- cron 报告必须标注数据源和时间戳。
