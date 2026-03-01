# 办公自动化套件

知识工作者每天花 2-3 小时在重复性办公任务上：筛选邮件、整理文件、写会议纪要、编周报。每项单独不难，但加起来消耗大量时间和精力。

这个用例把日常办公中最常见的重复性工作交给 OpenClaw，让它按照你的偏好自动处理。

## 它能做什么

| 场景 | 效果 |
|------|------|
| **邮件管理** | 定时检查收件箱，自动分类、摘要重要邮件、按规则回复 |
| **文件整理** | 按类型/日期/项目自动分类下载文件夹中的文件 |
| **会议纪要** | 会议结束后自动生成结构化纪要，推送到群聊 |
| **周报生成** | 汇总本周工作邮件和任务进展，生成格式化周报 |
| **日程同步** | 从微信截图中提取约会信息，自动创建日历事件 |

## 所需技能

根据你的邮箱选择：

- 国内邮箱（163/QQ）：[imap-smtp-email](https://playbooks.com/skills/openclaw/skills/imap-smtp-email) —— 支持标准 IMAP/SMTP 协议，兼容 163、QQ 邮箱等国内服务
- Gmail：[gog](https://docs.openclaw.ai/tools/skills) —— OpenClaw 官方维护的 Google Workspace 技能（需安装和配置 OAuth）
- Outlook/Microsoft 365：[outlook](https://playbooks.com/skills/openclaw/skills/outlook) —— 完整的 Outlook 邮件和日历管理（安装：`npx playbooks add skill openclaw/skills --skill outlook`）

## 如何设置

### 邮件自动化（以 163 邮箱为例）

1. 安装邮件技能：
```bash
npx playbooks add skill openclaw/skills --skill imap-smtp-email
```

2. 配置 163 邮箱的 IMAP/SMTP（需要在邮箱设置中开启 IMAP 并获取授权码）

> **安全提示**：邮箱授权码属于敏感凭证，请通过技能的 `.env` 文件配置，不要在对话中直接粘贴。确保 `.env` 已加入 `.gitignore`。

3. 设置定时任务：
```text
每天早上 9 点检查我的邮箱，把过去 24 小时的重要邮件摘要发给我。
筛选规则：忽略广告和订阅邮件，重点关注来自团队成员和客户的邮件。
把摘要保存到记忆中，方便我随时查看。
```

### 文件整理（提示词参考）

OpenClaw 内置文件系统工具可操作本地文件，执行前会请求确认。首次使用建议在测试文件夹中试运行：
```text
帮我把下载文件夹里的文件按类型分类：
- PDF 放到"文档"文件夹
- 图片放到"图片"文件夹
- 表格放到"数据"文件夹
超过 30 天未修改的文件移到"归档"文件夹。
```

### 周报生成（提示词参考）

```text
汇总我本周收发的工作邮件，提取关键项目进展和待办事项，生成一份周报。
格式：按项目分类，每个项目列出本周进展和下周计划。
```

## 实用建议

- **渐进式自动化**：先从一个场景开始（比如邮件摘要），跑通后再扩展到其他场景
- **设置偏好记忆**：让 OpenClaw 记住你的邮件分类偏好、周报格式、文件整理规则，随着使用逐渐优化
- **定时任务组合**：多个任务可以串联——早上 9 点邮件摘要 → 下午 5 点整理当天文件 → 周五 4 点生成周报

## 相关链接

- [imap-smtp-email 技能 - ClawHub](https://playbooks.com/skills/openclaw/skills/imap-smtp-email)
- [outlook 技能 - ClawHub](https://playbooks.com/skills/openclaw/skills/outlook)
- [CSDN - OpenClaw 办公自动化实战](https://blog.csdn.net/weixin_41194129/article/details/157644237)
