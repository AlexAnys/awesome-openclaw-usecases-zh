# AI 健身教练

> 含国内适配：Garmin 中国区 (garmin.cn) / 国产手表替代方案

你每天戴着 Garmin 手表跑步、骑车、游泳，积累了大量睡眠、心率变异性（HRV）、身体电量（Body Battery）等数据——但这些数据分散在 Garmin Connect 的各个页面里，你很难把它们串联成"今天该不该练"的决策。

这个工作流通过 MCP（模型上下文协议）将 Garmin 数据接入 OpenClaw，让 AI 成为你的私人教练：

- **晨间评估**：综合睡眠、HRV、静息心率、Body Battery 生成准备度评分（1-10），给出当日训练建议
- **训练复盘**：每次训练后分析心率区间、配速趋势、负荷变化，评估伤病风险
- **周度报告**：计算 ACWR（急性:慢性负荷比），追踪训练量趋势，规划下周安排
- **智能计划**：根据你的目标赛事和当前状态，生成自适应训练计划

## 它能做什么

| 指令 | 功能 | 输出 |
|------|------|------|
| `/fitness-coach morning` | 晨间准备度评估 | 准备度评分 + 当日计划 + 营养建议 |
| `/fitness-coach evening` | 训练复盘分析 | 训练分析 + 伤病评估 + 赛事信心指数 |
| `/fitness-coach weekly` | 周度回顾 | 训练量趋势 + ACWR 分析 + 下周规划 |
| `/fitness-coach plan` | 训练计划管理 | 创建或更新个性化训练方案 |
| `/fitness-coach` | 自由对话 | 训练、营养、赛事策略讨论 |

### 核心算法

**准备度评分（Readiness Score，1-10）**

以 5 分为基准，根据多维度数据动态调整：

| 因素 | 调整幅度 |
|------|---------|
| 睡眠质量 | ±2 分 |
| HRV（心率变异性）状态 | +1 至 -2 分 |
| Body Battery（身体电量）充电量 | ±2 分 |
| 静息心率趋势 | ±1 分 |
| 当前伤病状态 | -1 分 |

**ACWR 监控（Acute:Chronic Workload Ratio，急性:慢性负荷比）**

- ATL（急性训练负荷）：7 天平均训练负荷
- CTL（慢性训练负荷）：28 天平均训练负荷
- 安全区间：0.8–1.3，超出此范围提示过度训练或训练不足风险

**赛事信心指数（Race Confidence Score，0-100%）**

```
信心指数 = 伤病状态(40%) + 训练负荷完成度(25%) + 体能水平(25%) + 恢复质量(10%)
```

- 伤病状态：疼痛等级转换为百分比（无伤=100%，严重疼痛 4+/10=20%）
- 训练负荷完成度：每周计划完成率
- 体能水平：课程完成率(40%) + 阈值训练质量(30%) + 长距离效率(30%)
- 恢复质量：7 天 HRV 趋势分析

## 所需技能

- [Garmin MCP Server](https://github.com/Taxuspt/garmin_mcp)（96+ 工具，MIT 许可，281 stars） ⭐⭐
- [Coach Paddy 技能文件](https://github.com/iflow-mcp/borisbw-claude-fitness-cn)（slash 命令定义）⭐
- Garmin Connect 账号 + 兼容手表（Forerunner / Fenix / Venu 系列等）
- 可选：[Obsidian](https://obsidian.md/) 用于持久化训练记忆

> 难度说明：⭐ 入门级 ⭐⭐ 需要命令行基础 ⭐⭐⭐ 需要开发经验

## 如何设置

### 第一步：安装 Garmin MCP Server

预认证（仅需一次，处理 MFA 多因素认证）：

```bash
# 国际版
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth

# 中国区（garmin.cn 用户）
GARMIN_IS_CN=true uvx --python 3.12 --from git+https://github.com/BorisBW/garmin-mcp-cn garmin-mcp-auth
```

按提示输入 Garmin 账号邮箱、密码和 MFA 验证码。认证令牌（OAuth token）将保存到 `~/.garminconnect`。

### 第二步：配置 MCP

在 `~/.claude/settings.json` 中添加 Garmin MCP Server：

```jsonc
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python", "3.12",
        "--from", "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ]
    }
  }
}
```

中国区用户使用以下配置：

```jsonc
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python", "3.12",
        "--from", "git+https://github.com/BorisBW/garmin-mcp-cn",
        "garmin-mcp"
      ],
      "env": {
        "GARMIN_IS_CN": "true"
      }
    }
  }
}
```

> 注意：不要在配置文件中硬编码邮箱和密码。使用预认证步骤保存的令牌，或通过环境变量 `GARMIN_EMAIL` 和 `GARMIN_PASSWORD` 传递凭证。

### 第三步：安装 Coach Paddy 技能

```bash
# 克隆技能仓库
git clone https://github.com/iflow-mcp/borisbw-claude-fitness-cn.git

# 将技能文件复制到 OpenClaw 命令目录
cp borisbw-claude-fitness-cn/fitness-coach.md ~/.claude/commands/
```

### 第四步（可选）：配置 Obsidian 记忆

Coach Paddy 支持将训练数据持久化到 Obsidian 笔记，实现跨会话记忆：

```
你的 Obsidian 库/
├── Fitness/
│   ├── Coach Memory.md      # 运动员档案与历史
│   ├── Training Plan.md     # 当前周训练计划
│   ├── Recovery Log.md      # 每日恢复指标记录
│   └── Logs/                # 历史周度报告归档
```

在 `~/.claude/commands/fitness-coach.md` 中更新 Obsidian 库路径，指向你的实际目录。

### 第五步：开始使用

```text
/fitness-coach morning
```

OpenClaw 将通过 Garmin MCP 拉取你最近的睡眠、HRV、Body Battery 等数据，计算准备度评分，并给出今日训练建议。

## 实用建议

- **数据同步**：确保 Garmin 手表在使用前已同步到 Garmin Connect，MCP 读取的是云端数据
- **首次使用**：先运行 `/fitness-coach`，告诉教练你的基本信息（年龄、目标赛事、伤病史、心率区间），这些会保存到 Obsidian 记忆中
- **ACWR 解读**：比值 < 0.8 说明训练不足，> 1.3 提示过度训练风险，0.8–1.3 是最佳区间
- **赛前减量**：比赛前 2-3 周，教练会自动建议减量（taper），注意观察 ACWR 从高位回落
- **MFA 令牌过期**：如果 MCP 连接失败，重新运行 `garmin-mcp-auth` 刷新令牌
- **隐私提示**：所有数据通过 MCP 在本地处理，不经过第三方服务器（除 Garmin Connect 官方 API）

## 中国用户适配

### Garmin 中国区 vs 国际版

| 维度 | 国际版 (garmin.com) | 中国区 (garmin.cn) |
|------|--------------------|--------------------|
| 数据端点 | connect.garmin.com | connect.garmin.cn |
| 账号体系 | 全球 Garmin 账号 | 中国区独立账号 |
| 地图服务 | Google Maps | 百度地图 |
| MCP 服务器 | Taxuspt/garmin_mcp | BorisBW/garmin-mcp-cn |
| 环境变量 | 无需额外设置 | `GARMIN_IS_CN=true` |
| 工具覆盖 | 96+ 工具 | 96+ 工具（完全一致） |

两个平台的数据**完全隔离**，中国区账号无法通过国际版端点访问数据，反之亦然。使用前确认你的 Garmin Connect 账号注册在哪个平台。

### 国产运动手表替代方案

如果你使用华为、小米等国产手表，目前没有成熟的 MCP Server 可以直接替代，但有以下路径：

| 品牌 | 数据获取方式 | 可行性 |
|------|------------|--------|
| **华为** | [HUAWEI Health Kit](https://developer.huawei.com/consumer/cn/hms/huaweihealth/) REST API | 需申请开发者权限，API 覆盖运动、睡眠、心率等 |
| **小米/Amazfit** | [华米开放平台](https://huami.gitbooks.io/iot/content/) API | 支持活动、睡眠、心率数据读取，需开发者授权 |
| **通用方案** | 导出数据为 `.fit` / `.tcx` / `.gpx` 文件，手动提供给 AI | 最通用但需手动操作 |

> 现状：截至 2026 年 3 月，国产手表尚无社区维护的 MCP Server。如果你有开发能力，可以参考 Garmin MCP 的架构，基于华为 Health Kit REST API 封装一个 MCP Server——这是一个有价值的开源贡献方向。

### 推送渠道适配

| 原版方案 | 国内替代 | 说明 |
|---------|---------|------|
| Obsidian 本地笔记 | **语雀 / Notion** | 国内访问更稳定 |
| 无推送（按需查询） | **钉钉群机器人** | 配合 cron job 实现定时晨间推送 |
| 无推送（按需查询） | **飞书机器人** | 支持富文本卡片，适合格式化报告 |

### 提示词适配

原版技能文件为中英双语，可直接使用。如果需要纯中文输出，在首次对话中补充：

```text
/fitness-coach
请用中文回复所有内容。我的 Garmin 账号在中国区 (garmin.cn)，
手表型号是 Forerunner 265。目标是今年完成一个半程马拉松，
当前周跑量约 30 公里。无伤病史。
```

## 相关链接

- [Garmin MCP Server](https://github.com/Taxuspt/garmin_mcp) — 96+ 工具，MIT 许可，支持全部 Garmin Connect 数据
- [Garmin MCP 中国区 Fork](https://github.com/BorisBW/garmin-mcp-cn) — garmin.cn 适配版本
- [Coach Paddy 技能](https://github.com/iflow-mcp/borisbw-claude-fitness-cn) — AI 健身教练 slash 命令定义
- [Garmin Training Readiness 官方说明](https://www.garmin.com/en-US/garmin-technology/running-science/physiological-measurements/training-readiness/) — Garmin 原生准备度算法介绍
- [HUAWEI Health Kit](https://developer.huawei.com/consumer/cn/hms/huaweihealth/) — 华为运动健康开发者平台

---

**灵感来源**：[BorisBW/claude-fitness-cn](https://github.com/BorisBW/claude-fitness-cn)、[Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)
