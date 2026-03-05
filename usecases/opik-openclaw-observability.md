# OpenClaw + Opik 可观测性追踪

当 OpenClaw 开始承载真实业务后，问题通常出在「看不见」：一次对话会触发 LLM 调用、工具调用、子智能体协作，但排障信息分散在不同日志里，定位成本很高。

这个用例演示如何通过 `@opik/opik-openclaw` 把 OpenClaw 运行过程接入 Opik，实现统一追踪、错误排查和成本监控。

## 痛点

- **链路不可见**：很难把一次用户请求和后续全部执行步骤关联起来
- **故障排查慢**：排查需要在网关日志、模型日志、工具日志之间来回切换
- **成本不透明**：按任务/工作流观察 token 和成本的能力不足

## 功能说明

- 将 OpenClaw 事件导出为 Opik 中的 trace/span
- 覆盖以下关键数据：
  - LLM 请求与响应 span
  - 工具调用 span（入参/结果/错误）
  - 子智能体生命周期 span
  - 运行结束元数据
  - 用量与成本元数据

## 提示词（可直接复制）

```text
请做一次网关健康检查，发送一条测试消息，并汇总今天 Opik 里所有失败的工具调用及错误原因。
```

```text
请统计最近 24 小时 Opik 追踪里 token 和成本最高的工作流，并给出每个工作流 1 条优化建议。
```

## 所需技能

- 基础 OpenClaw CLI 使用能力
- 无需额外 OpenClaw skills

## 配置方法

1. 安装插件：

```bash
openclaw plugins install @opik/opik-openclaw
```

2. 运行配置向导：

```bash
openclaw opik configure
```

3. 检查生效配置：

```bash
openclaw opik status
```

4. 发送测试请求并生成追踪：

```bash
openclaw gateway run
openclaw message send "hello from openclaw"
```

5. 在 Opik 中确认 trace 与 span 正常入库。

## 相关链接

- [opik-openclaw 仓库](https://github.com/comet-ml/opik-openclaw)
- [Opik 文档](https://www.comet.com/docs/opik/)
- [OpenClaw 插件文档](https://docs.openclaw.ai/plugin)
