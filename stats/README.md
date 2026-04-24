# 流量统计

此目录由 `.github/workflows/traffic-snapshot.yml` 自动维护，每日北京时间 09:00 抓取。

## 文件

- `latest.md` — 最新一天的快照（人类可读）
- `history.jsonl` — 历史快照流（每行一个 JSON，用于趋势分析）

## 关键指标

| 指标 | 来源 | 用途 |
|------|------|------|
| stars / forks / watchers | `/repos/:owner/:repo` | 健康度基线 |
| views.count / uniques | `/traffic/views` | 14 天访问量（unique 更准） |
| clones.count / uniques | `/traffic/clones` | 有多少人把仓库拉到本地 |
| referrers | `/traffic/popular/referrers` | 外部站点引流来源 |
| paths | `/traffic/popular/paths` | 具体哪些用例被访问——**关键词流量主信号** |

## 用途

- 对比做分发动作前后的 views/uniques 变化，判断渠道 ROI
- 跟踪 paths 变化——当"claude-code 相关文件"开始出现在 top 10，说明跨 agent 定位生效
- referrers 新出现的域名是发现新引流源的机会
