# Hermes 适配：用 Lark CLI 让 Agent 操作飞书

> 这是 [用 Lark CLI 让 Agent 操作飞书](../../../usecases/cn-feishu-lark-cli.md) 的 Hermes 执行差异说明。完整背景、痛点和原始提示词请先阅读原用例。

## 一句话适配结论

`partial`。Hermes 可以通过 terminal 调用 Lark CLI；安装和 `doctor` 可验证。完整业务操作需要浏览器 OAuth 和飞书权限，所有写操作应默认 `--dry-run`。

## 概念映射

| OpenClaw | Hermes | 备注 |
|---|---|---|
| Agent Skills 自动 symlink | 外部 CLI + Hermes 指令约束 | Hermes 不应假设 Claude/OpenClaw skill 自动加载 |
| 飞书用户身份 | Lark CLI OAuth | 首次授权通常需要浏览器 |
| 发消息/改表格 | `lark-cli ... --dry-run` 先预演 | 用户确认后再执行 |

## Hermes 下的执行差异

1. **安装隔离**：可用临时 npm prefix 或用户级 npm，避免污染全局环境。
2. **授权限制**：headless 服务器无法独立完成首次 OAuth，需在本机或可打开浏览器的环境完成。
3. **输出格式**：让 Hermes 默认使用 `--format json` 解析，给人看时再用 `--format pretty`。
4. **副作用命令**：发消息、建群、改多维表格、改日程前必须 `--dry-run`。

## 最小验证

```bash
npm view @larksuite/cli version repository.url license
export npm_config_prefix="$HOME/.local/lark-cli-test"
export PATH="$npm_config_prefix/bin:$PATH"
npm install -g @larksuite/cli
lark-cli --help
lark-cli doctor
```

期望：`--help` 列出 docs/base/calendar/im 等命令；未授权时 `doctor` 提示运行 `lark-cli config init`。

## 已知限制 / 风险

- `credential_heavy`：OAuth token 有有效期，不能提交到仓库。
- `external_write`：飞书写操作可能影响真实团队空间，默认 dry-run。
- 部分企业权限需管理员审批。
