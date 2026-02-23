# 自愈式家庭服务器与基础设施管理

运行家庭服务器意味着 7x24 小时随时待命处理自己的基础设施问题。服务在凌晨 3 点宕机、证书默默过期、磁盘空间耗尽、Pod 进入崩溃循环（crash-loop）——而这些都发生在你睡觉或外出的时候。

这个用例将 OpenClaw 变成一个持久运行的基础设施智能体（agent），拥有 SSH 访问权限、自动化定时任务（cron job），以及在你发现问题之前就检测、诊断和修复问题的能力。

## 痛点

家庭实验室运维者和自托管用户面临持续的维护负担：

- 健康检查、日志监控和告警需要手动设置和持续关注
- 当出现故障时，你必须 SSH 登录、诊断并修复——通常还是在手机上操作
- 基础设施即代码（Infrastructure-as-Code）工具（Terraform、Ansible、Kubernetes 清单）需要定期更新
- 关于你的配置的知识存储在你的脑子里，而不是可搜索的文档中
- 日常任务（邮件分类、部署检查、安全审计）每周要消耗数小时

## 功能概述

- **自动化健康监控**：基于定时任务检查服务、部署和系统资源
- **自我修复**：通过健康检查检测问题，并自主应用修复（重启 Pod、扩展资源、修复配置）
- **基础设施管理**：编写并应用 Terraform、Ansible 和 Kubernetes 清单
- **晨间简报**：每日系统健康状况、日历、天气和任务板状态摘要
- **邮件分类**：扫描收件箱，标记可操作项目，归档噪音
- **知识提取**：将笔记和对话导出处理为结构化、可搜索的知识库
- **博客发布管线**：草稿 → 生成横幅图 → 发布到 CMS → 部署到托管平台——全自动化
- **安全审计**：定期扫描硬编码密钥、特权容器和过度宽松的访问权限

## 所需技能

- `ssh` 访问家庭网络机器
- `kubectl`，用于 Kubernetes 集群管理
- `terraform` 和 `ansible`，用于基础设施即代码
- `1password` CLI，用于密钥管理
- `gog` CLI，用于邮件访问
- 日历 API 访问
- Obsidian vault 或笔记目录（用于知识库）
- `openclaw doctor`，用于自我诊断

## 如何设置

### 1. 核心智能体配置

为你的智能体命名并在 AGENTS.md 中定义其访问范围：

```text
## Infrastructure Agent

You are Reef, an infrastructure management agent.

Access:
- SSH to all machines on the home network (192.168.1.0/24)
- kubectl for the K3s cluster
- 1Password vault (read-only for credentials, dedicated AI vault)
- Gmail via gog CLI
- Calendar (yours + partner's)
- Obsidian vault at ~/Documents/Obsidian/

Rules:
- NEVER hardcode secrets — always use 1Password CLI or environment variables
- NEVER push directly to main — always create a PR
- Run `openclaw doctor` as part of self-health checks
- Log all infrastructure changes to ~/logs/infra-changes.md
```

### 2. 自动化定时任务系统

此配置的核心在于计划任务系统。在 HEARTBEAT.md 中配置：

```text
## Cron Schedule

Every 15 minutes:
- Check kanban board for in-progress tasks → continue work

Every hour:
- Monitor health checks (Gatus, ArgoCD, service endpoints)
- Triage Gmail (label actionable items, archive noise)
- Check for unanswered alerts or notifications

Every 6 hours:
- Knowledge base data entry (process new Obsidian notes)
- Self health check (openclaw doctor, disk usage, memory, logs)

Every 12 hours:
- Code quality and documentation audit
- Log analysis via Loki/monitoring stack

Daily:
- 4:00 AM: Nightly brainstorm (explore connections between notes)
- 8:00 AM: Morning briefing (weather, calendars, system stats, task board)
- 1:00 AM: Velocity assessment (process improvements)

Weekly:
- Knowledge base QA review
- Infrastructure security audit
```

### 3. 安全设置（关键）

这是不可妥协的。在给你的智能体 SSH 访问权限之前：

```text
## Security Checklist

1. Pre-push hooks:
   - Install TruffleHog or similar secret scanner on ALL repositories
   - Block any commit containing hardcoded API keys, tokens, or passwords

2. Local-first Git workflow:
   - Use Gitea (self-hosted) for private code before pushing to public GitHub
   - CI scanning pipeline (Woodpecker or similar) runs before any public push
   - Human review required before main branch merges

3. Defense in depth:
   - Dedicated 1Password vault for AI agent (limited scope)
   - Network segmentation for sensitive services
   - Daily automated security audits checking for:
     * Privileged containers
     * Hardcoded secrets in code or configs
     * Overly permissive file/network access
     * Known vulnerabilities in deployed images

4. Agent constraints:
   - Branch protection: PR required for main, agent cannot override
   - Read-only access where write isn't needed
   - All changes logged and auditable via git
```

### 4. 晨间简报模板

```text
## Daily Briefing Format

Generate and deliver at 8:00 AM:

### Weather
- Current conditions and forecast for [your location]

### Calendars
- Your events today
- Partner's events today
- Conflicts or overlaps flagged

### System Health
- CPU / RAM / Storage across all machines
- Services: UP/DOWN status
- Recent deployments (ArgoCD)
- Any alerts in last 24h

### Task Board
- Cards completed yesterday
- Cards in progress
- Blocked items needing attention

### Highlights
- Notable items from nightly brainstorm
- Emails requiring action
- Upcoming deadlines this week
```

## 关键洞察

- **"没想到我竟然有了一台自愈式服务器"**：智能体可以运行 SSH、Terraform、Ansible 和 kubectl 命令，在你甚至还不知道有问题的时候就修复基础设施故障
- **AI 会硬编码密钥**：这是头号安全风险。如果你不设置防护措施，智能体会毫不犹豫地将 API 密钥内联写入代码。Pre-push 钩子和密钥扫描是必须的
- **本地优先的 Git 流程至关重要**：永远不要让智能体直接推送到公共仓库。使用私有 Gitea 实例作为暂存区，并配合 CI 扫描
- **定时任务才是真正的产品**：计划自动化（健康检查、邮件分类、简报）比临时命令提供更多的日常价值
- **知识提取具有复利效应**：将笔记、对话导出和邮件处理为结构化知识库，随着时间推移会越来越有价值——一位用户仅从 ChatGPT 历史记录中就提取了 49,079 个原子事实

## 灵感来源

这个用例基于 Nathan 的详细文章 ["Everything I've Done with OpenClaw (So Far)"](https://madebynathan.com/2026/02/03/everything-ive-done-with-openclaw-so-far/)，他在文中描述了自己的 OpenClaw 智能体"Reef"运行在家庭服务器上，拥有所有机器的 SSH 访问权限、一个 Kubernetes 集群、1Password 集成，以及一个包含 5,000+ 笔记的 Obsidian vault。Reef 运行着 15 个活跃的定时任务、24 个自定义脚本，并自主构建和部署了多个应用，包括一个任务管理 UI。Nathan 在第一天就遭遇 API 密钥泄露后得到的惨痛教训："AI 助手会毫不犹豫地硬编码密钥。它们有时候没有人类那样的安全直觉。"他的纵深防御安全设置（TruffleHog pre-push 钩子、本地 Gitea、CI 扫描、每日审计）对于任何尝试此模式的人来说都是必读内容。

同样在 [OpenClaw Showcase](https://openclaw.ai/showcase) 上，`@georgedagg_` 描述了类似的模式：部署监控、日志审查、配置修复和提交 PR——所有这些都是在遛狗的时候完成的。

## 相关链接

- [Nathan 的完整文章](https://madebynathan.com/2026/02/03/everything-ive-done-with-openclaw-so-far/)
- [OpenClaw 文档](https://github.com/openclaw/openclaw)
- [TruffleHog（密钥扫描）](https://github.com/trufflesecurity/trufflehog)
- [K3s（轻量级 Kubernetes）](https://k3s.io/)
- [Gitea（自托管 Git）](https://gitea.io/)
- [n8n（工作流自动化）](https://n8n.io/)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/self-healing-home-server.md)
