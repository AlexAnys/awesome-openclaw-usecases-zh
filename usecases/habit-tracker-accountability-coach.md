# 习惯追踪与打卡教练

市面上的习惯打卡 App 你大概都试过：开头三天兴致勃勃，一周后就再也不打开了。问题不在 App，而在于打卡本身是被动的——没人催你，你就忘了。

这个用例把 OpenClaw 变成你的专属打卡教练：每天定时主动来找你，问你今天练了没、读了没；根据你的连击和掉线情况调整语气，状态好就夸，掉线就换着花样 nudges 你。不用麻烦朋友监督，也不靠意志力硬撑。

> 改编自英文原版 [Habit Tracker & Accountability Coach](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/habit-tracker-accountability-coach.md)，国内消息通道适配。

## 它能做什么

- **每日主动打卡提醒**：按你设定的时间（比如早上 7 点问晨间习惯、晚上 9 点复盘）通过飞书 / 钉钉 / 企业微信来找你
- **习惯管理**：自定义习惯——运动、阅读、冥想、喝水、写代码，随你定
- **连击统计**：记住每个习惯的连续打卡天数，消息里直接告诉你"这是你连续第 12 天"
- **自适应提醒**：状态好时鼓励，连续掉线时换种方式 persistent 地提醒，不烦人但有效
- **每周复盘**：生成周报——完成率、最长连击、规律发现（比如"你每周三最容易跳过锻炼"）

## 所需技能

- 消息通道：飞书 / 钉钉 / 企业微信（任选其一，用于收发打卡消息）
- 定时任务：OpenClaw 内置 cron / Heartbeat（用于每日提醒和每周复盘）

## 如何设置

### 第一步：接入消息通道

以飞书为例（钉钉 / 企业微信流程类似），参考 [飞书 AI 助手](cn-feishu-ai-assistant.md) 的接入步骤：

```bash
# 添加飞书渠道（交互式引导）
openclaw channels add
# 选择 Feishu → 粘贴 App ID → 粘贴 App Secret

# 重启网关
openclaw gateway restart
```

### 第二步：定义你的习惯

在 OpenClaw 的配置或 MEMORY 里写下习惯清单，例如：

- 每天跑步 30 分钟
- 每天阅读 30 分钟
- 每天冥想 10 分钟

### 第三步：配置每日打卡提醒

用 OpenClaw 的定时任务（cron）创建两条规则：早上 7:00 询问早间习惯打卡，晚上 21:00 复盘当天完成情况。让 agent 在每次对话后更新连击数据（存本地 JSON 或 SQLite 即可），并根据连续打卡 / 掉线天数调整话术。

### 第四步：配置每周复盘

每周日 21:00 生成习惯周报：完成率、最长连击、掉线规律，一次性推送给你。

## 实用建议

- **从 1-2 个习惯开始**：贪多是打卡失败的首要原因，先让链条转起来
- **提醒时间贴着生活节奏**：晨间习惯配早上提醒，复盘配晚上，别在开会时间打扰
- **允许"补卡"**：直接回复"昨天跑了"，手动补录不断连击
- **掉线 3 天自动降级**：让 agent 在连续掉线时主动问"要不要把目标调小一点"，比硬撑更容易坚持

## 相关链接

- 英文原版：[Habit Tracker & Accountability Coach](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/habit-tracker-accountability-coach.md)
- 消息通道接入：[飞书 AI 助手](cn-feishu-ai-assistant.md)
