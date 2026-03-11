# M365 邮件分拣：Prefetch 两阶段架构

> 含国内适配：QQ 企业邮箱 / 网易企业邮箱 / M365 世纪互联版

跨多个 Microsoft 365 租户的智能邮件监控，通过成本优化的预取（prefetch）架构将 token 消耗降低 99%，再由智能分拣决定哪些邮件值得提醒你——哪些可以等一等。

## 痛点

如果你用 OpenClaw 监控邮件，大概率踩过这些坑：

- **成本爆炸**：每次 cron job 触发都会写入一次完整的系统提示词缓存。如果每 15 分钟检查一次邮件，一天就是 96 次冷启动——智能体还没读邮件就已经开始烧钱了。用 Claude Sonnet 每次冷启动约 $0.20，光邮件检查每天就要 $20。
- **Token 膨胀**：如果智能体直接通过 mcporter 调用 M365 工具，会拉取完整的 HTML 邮件正文——CSS、追踪像素、图片，全部塞进来。一次收件箱检查可能消耗 145 万 token。你的智能体在吃整个渲染后的网页当早餐。
- **提醒疲劳**：没有智能分拣，每封邮件都会弹通知——营销邮件、新闻订阅、自动回执、过期消息——你很快就不再信任这些提醒了。

## 解决方案

两阶段架构，将**数据收集**（便宜，无 LLM 参与）和**智能分拣**（LLM 参与，但输入 token 极少）分离开来。

**第一阶段 — 预取（Prefetch，系统 cron，零 AI 成本）：**
一个 Python 脚本在工作时间每 30 分钟运行一次。它调用 mcporter 从多个 M365 租户拉取邮件元数据，去除回复链引用，按时间窗口过滤，然后写入一个小 JSON 文件。成本：零 LLM token。

**第二阶段 — 分拣（Triage，OpenClaw cron，极少 token）：**
7 分钟后，你的 OpenClaw 智能体读取预构建的 JSON 文件，做出判断：什么是紧急的、什么是常规的、什么可以等到早间简报再处理。它通过 Telegram 发送带表情分类和建议操作的提醒。成本：约 1.4 万 token，而非 145 万。

## 所需技能

- **mcporter**（内置） — Microsoft 365 集成
- **message** 或 Telegram 频道 — 用于发送提醒

## 如何设置

### 第一阶段：预取脚本

创建一个 Python 脚本，作为系统 cron job 运行（不是 OpenClaw cron——无 LLM 成本）：

```python
#!/usr/bin/env python3
"""邮件预取 — 通过系统 cron 运行，将 JSON 写入供 OpenClaw 读取。"""
import subprocess, json, os, tempfile
from datetime import datetime, timedelta, timezone
from zoneinfo import ZoneInfo  # Python 3.9+，自动处理夏令时

CST = ZoneInfo("America/Chicago")
LOOKBACK = timedelta(minutes=35)  # 比 30 分钟间隔稍宽一些
OUTPUT = "/tmp/openclaw/email-monitor-pending.json"

TENANTS = [
    {"name": "Primary", "account": "user@company.com"},
    {"name": "Secondary", "account": "user@otherdomain.com"},
]

def fetch_tenant(tenant):
    since = (datetime.now(CST) - LOOKBACK).astimezone(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")
    cmd = [
        "openclaw", "skills", "call", "mcporter",
        "--", "outlook", "mail", "list",
        "--account", tenant["account"],
        "--filter", f"receivedDateTime ge {since}",
        "--select", "subject,from,receivedDateTime,isRead,bodyPreview",
        "--top", "20",
        "--output", "text"
    ]
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    except subprocess.TimeoutExpired:
        return []
    if result.returncode != 0:
        return []
    try:
        return json.loads(result.stdout)
    except json.JSONDecodeError:
        return []

def strip_reply_chains(emails):
    """去除 bodyPreview 中的回复链引用内容以节省 token。"""
    for email in emails:
        preview = email.get("bodyPreview", "")
        for marker in ["From:", "-----Original", "On ", "> "]:
            idx = preview.find(marker)
            if idx > 50:  # 至少保留 50 个字符
                preview = preview[:idx].rstrip()
                break
        email["bodyPreview"] = preview[:200]  # 截断到 200 字符
    return emails

results = {}
for tenant in TENANTS:
    emails = fetch_tenant(tenant)
    results[tenant["name"]] = strip_reply_chains(emails)

os.makedirs(os.path.dirname(OUTPUT), exist_ok=True)
tmp_output = None
try:
    fd, tmp_output = tempfile.mkstemp(dir=os.path.dirname(OUTPUT), suffix=".json")
    with os.fdopen(fd, "w") as f:
        json.dump({"fetched_at": datetime.now(CST).isoformat(), "tenants": results}, f)
    os.replace(tmp_output, OUTPUT)
except (OSError, IOError) as e:
    print(f"ERROR: Failed to write {OUTPUT}: {e}")
    if tmp_output and os.path.exists(tmp_output):
        os.remove(tmp_output)
```

**系统 crontab**（每 30 分钟运行一次，早 8 点到晚 11 点）：
```bash
CRON_TZ=America/Chicago
*/30 8-23 * * * /usr/bin/python3 /path/to/email-prefetch.py >> /tmp/openclaw/email-monitor.log 2>&1
```

### 第二阶段：OpenClaw 定时任务

创建一个 OpenClaw cron job，在预取之后 7 分钟运行：

```bash
openclaw cron add \
  --name "email-monitor" \
  --schedule "7,37 8-23 * * *" \
  --message "Read /tmp/openclaw/email-monitor-pending.json. Triage emails using EMAIL_MONITOR_PREFS.md rules. Alert me via Telegram only for items that need my attention. Use emoji categories and add → action lines for actionable items. Add a blank line between items. If nothing is actionable, skip the alert entirely."
```

### 第三阶段：分拣规则

创建 `~/.openclaw/workspace/EMAIL_MONITOR_PREFS.md`：

```markdown
# 邮件监控偏好设置

## 始终提醒（无论什么时间）
- 金融交易、支付失败、银行提醒
- 任何服务的安全警报
- VIP 联系人（见 VIP_CONTACTS.md）
- 来自学校、托管机构关于孩子的消息

## 工作时间提醒
- 需要回复的客户邮件
- 日历邀请
- 服务中断通知

## 抑制 / 仅在早间简报中展示
- 营销邮件和新闻订阅
- 自动回执和确认信
- 社交媒体通知
- 超过 6 小时的过期消息（不值得深夜提醒）

## 格式规范
- 🔴 紧急：需要 1 小时内回复
- 🟡 可操作：今天需要回复
- 🔵 信息性：仅供了解，无需操作
- → 每个可操作项附带建议操作
```

创建 `~/.openclaw/workspace/VIP_CONTACTS.md`：
```markdown
# VIP 联系人 — 始终置顶
- boss@company.com（经理）
- spouse@email.com（家人）
- school@district.edu（孩子学校）
```

## mcporter 踩坑记录

这些问题花了原作者好几天来调试，记录在此帮你避坑：

1. **始终使用 `--select` 配合 `bodyPreview`**，而不是拉取完整邮件正文。不加这个参数，mcporter 会返回渲染后的 HTML——CSS、图片、追踪像素——间歇性地破坏 JSON 解析，并且消耗 100 倍的 token。

2. **始终使用 `--output text`** 以获取干净的 JSON。默认输出会将响应包装在 MCP 信封中，需要额外解析。

3. **日历日期过滤是静默失效的。** mcporter 的 `startdatetime` / `enddatetime` 参数实际上不会过滤日历事件。需要先检索所有事件，然后在 Python 中做后置过滤。

## 效果对比

| 指标 | 改造前（直接 API 调用） | 改造后（Prefetch） |
|------|----------------------|-------------------|
| 每次检查 token | ~1,450,000 | ~14,000 |
| JSON 解析失败 | 间歇性发生 | 零 |
| 每日成本（仅邮件） | ~$20+ | ~$1-2 |
| 误报率 | 高 | 低（VIP + 时间感知） |

## 实用建议

- **过期消息抑制**是最大的体验提升。如果一封邮件 6 小时前到达且现在是晚上 10 点，智能体应该跳过它而不是震动你的手机。它会出现在早间简报中。
- **回复链去除**比你想象的更重要。不去除的话，一个 10 条消息的邮件线程每次检查都会发送完整历史。将 `bodyPreview` 截断到 200 字符。
- **7 分钟偏移**确保 JSON 文件在智能体读取时始终是新鲜的。没有偏移的话，竞态条件会导致智能体读取到过期或写入中的数据。
- **多租户是免费的。** 预取脚本按顺序遍历租户。添加另一个 M365 账号只需在配置中加一行。

## 为什么用 Prefetch 而不是直接工具调用？

每次 OpenClaw cron job 触发都会写入一次完整的系统提示词缓存（典型工作区使用 Claude Sonnet 约 $0.20）。这是唤醒智能体的固定成本。如果智能体随后再发起 3-4 次 mcporter 工具调用来检查多个收件箱，每次调用都会增加往返延迟和 token 消耗。

Prefetch 模式反转了这个流程：数据收集零成本（Python + 系统 cron），然后给智能体一个预处理好的 JSON 文件去推理。一次缓存写入，一次文件读取，一次分拣决策。智能体的工作是判断，而不是数据收集。

## 出处

由 Triss Manifold 项目开发——一个运行在 Beelink SER5 迷你 PC 上的生产级 OpenClaw 部署，为一家托管服务提供商管理多个 M365 租户的邮件分拣。自 2026 年 2 月起每日运行。

---

## 中国用户适配

国内企业邮件生态与 M365 差异较大。以下是针对国内环境的完整适配方案，核心架构（Prefetch 两阶段分离）完全适用，只需替换数据收集层。

### 国内邮件系统选择矩阵

| 邮件系统 | 协议支持 | 接入难度 | 适用场景 | 推荐度 |
|---------|---------|---------|---------|-------|
| **M365 世纪互联版** | Graph API（需改端点） | 中 | 外企/跨国团队 | 推荐 |
| **腾讯企业邮箱** | IMAP/SMTP + 开放 API | 低 | 企业微信生态用户 | 推荐 |
| **网易企业邮箱** | IMAP/SMTP + REST API | 低 | 传统企业用户 | 推荐 |
| **阿里企业邮箱** | IMAP/SMTP | 低 | 钉钉生态用户 | 可用 |

### M365 世纪互联版适配

如果你的企业使用世纪互联运营的 Microsoft 365（国内合规版本），mcporter 需要修改 API 端点才能正常工作。

**端点差异对照：**

| 服务 | 国际版 | 世纪互联版 |
|------|--------|-----------|
| Graph API | `graph.microsoft.com` | `microsoftgraph.chinacloudapi.cn` |
| 认证端点 | `login.microsoftonline.com` | `login.chinacloudapi.cn` |
| Outlook API | `outlook.office.com` | `partner.outlook.cn` |

**配置方式**：在 Azure Entra ID（世纪互联版）中注册应用时，需确保：
1. 在 [Azure 中国门户](https://portal.azure.cn/) 注册应用（不是 portal.azure.com）
2. 授予 `Mail.Read`、`Mail.ReadWrite` 等 Graph API 权限
3. mcporter 配置中将 Graph API 端点改为 `microsoftgraph.chinacloudapi.cn`

> **注意**：世纪互联版的 Graph API 功能可能滞后于国际版，使用前请查阅 [Azure 中国文档](https://docs.azure.cn/) 确认所需接口是否可用。

### QQ 企业邮箱 / 腾讯企业邮箱适配

腾讯企业邮箱（exmail.qq.com）与企业微信深度绑定，是国内使用量最大的企业邮箱之一。

**接入方式：IMAP 协议**

腾讯企业邮箱支持标准 IMAP 协议，无需专属 SDK，Python 标准库 `imaplib` 即可接入：

```python
#!/usr/bin/env python3
"""腾讯企业邮箱预取 — 通过 IMAP 协议获取邮件元数据。"""
import imaplib
import email
from email.header import decode_header
import json, os, tempfile
from datetime import datetime, timedelta

# 通过环境变量传递凭证，绝不硬编码
IMAP_SERVER = "imap.exmail.qq.com"
IMAP_PORT = 993
USERNAME = os.environ["QQ_EXMAIL_USER"]  # 例如 user@company.com
PASSWORD = os.environ["QQ_EXMAIL_PASSWORD"]  # 授权码，非登录密码

LOOKBACK_MINUTES = 35
OUTPUT = "/tmp/openclaw/email-monitor-pending.json"

def decode_mime_header(header_value):
    """解码 MIME 编码的邮件头。"""
    if not header_value:
        return ""
    decoded_parts = decode_header(header_value)
    result = []
    for part, charset in decoded_parts:
        if isinstance(part, bytes):
            result.append(part.decode(charset or "utf-8", errors="replace"))
        else:
            result.append(part)
    return "".join(result)

def fetch_emails():
    since = (datetime.now() - timedelta(minutes=LOOKBACK_MINUTES)).strftime("%d-%b-%Y")
    emails_data = []

    mail = imaplib.IMAP4_SSL(IMAP_SERVER, IMAP_PORT)
    try:
        mail.login(USERNAME, PASSWORD)
        mail.select("INBOX", readonly=True)

        # 搜索指定日期以来的邮件
        _, message_numbers = mail.search(None, f'(SINCE "{since}")')
        if not message_numbers[0]:
            return emails_data

        for num in message_numbers[0].split()[-20:]:  # 最多取最近 20 封
            _, msg_data = mail.fetch(num, "(RFC822.HEADER BODY.PEEK[TEXT]<0.500>)")
            if not msg_data or not msg_data[0]:
                continue

            raw_header = msg_data[0][1] if msg_data[0] else b""
            msg = email.message_from_bytes(raw_header)

            emails_data.append({
                "subject": decode_mime_header(msg.get("Subject", "")),
                "from": decode_mime_header(msg.get("From", "")),
                "receivedDateTime": msg.get("Date", ""),
                "bodyPreview": ""  # IMAP 预览需从正文截取
            })
    finally:
        mail.logout()

    return emails_data

results = {"腾讯企业邮箱": fetch_emails()}

os.makedirs(os.path.dirname(OUTPUT), exist_ok=True)
tmp_output = None
try:
    fd, tmp_output = tempfile.mkstemp(dir=os.path.dirname(OUTPUT), suffix=".json")
    with os.fdopen(fd, "w") as f:
        json.dump({"fetched_at": datetime.now().isoformat(), "tenants": results}, f, ensure_ascii=False)
    os.replace(tmp_output, OUTPUT)
except (OSError, IOError) as e:
    print(f"ERROR: Failed to write {OUTPUT}: {e}")
    if tmp_output and os.path.exists(tmp_output):
        os.remove(tmp_output)
```

**腾讯企业邮箱配置要点：**

1. 管理员在企业邮箱后台开启 IMAP 服务
2. 用户生成**客户端专用密码**（授权码），不使用登录密码
3. IMAP 服务器：`imap.exmail.qq.com`，端口 993（SSL）
4. API 调用频率限制：每 IP 不超过 10,000 次/分钟

> **安全提醒**：邮箱授权码属于敏感凭证，必须通过环境变量或 `.env` 文件传递，确保 `.env` 已加入 `.gitignore`。

### 网易企业邮箱适配

网易企业邮箱（qiye.163.com）同样支持 IMAP 协议，接入方式与腾讯类似。

**配置差异：**

| 配置项 | 腾讯企业邮箱 | 网易企业邮箱 |
|--------|------------|------------|
| IMAP 服务器 | `imap.exmail.qq.com` | `imap.qiye.163.com` |
| 端口 | 993（SSL） | 993（SSL） |
| 认证方式 | 客户端专用密码 | 客户端授权码 |
| API 文档 | [腾讯企业邮箱开放接口](https://www.qqbizmail.com/api/) | [网易企业邮箱接口文档](https://qiye.163.com/help/l-24.html) |

**接入步骤：**

1. 管理员发送邮件至 kf@office.163.com 申请开通集成接口功能
2. 获取 API 调用参数后，在邮箱设置中开启 IMAP 服务
3. 替换预取脚本中的 `IMAP_SERVER` 为 `imap.qiye.163.com`

网易企业邮箱还提供 REST API，支持邮件审核、备份、搜索等高级功能，适合有深度集成需求的场景。

### 推送渠道适配

原版通过 Telegram 推送分拣结果，国内用户可替换为以下方案：

| 原版方案 | 国内替代 | 接入方式 | 推荐场景 |
|---------|---------|---------|---------|
| Telegram | **企业微信群机器人** | Webhook，最简单 | 企业微信用户 |
| Telegram | **飞书群机器人** | Webhook，支持富文本卡片 | 飞书用户 |
| Telegram | **钉钉群机器人** | Webhook，支持 Markdown | 钉钉用户 |

**企业微信 Webhook 示例**（替换 OpenClaw 分拣阶段的推送部分）：

```text
分拣完邮件后，将结果通过企业微信群机器人 webhook 推送。
webhook 地址从环境变量 WECOM_WEBHOOK_URL 读取。
格式要求：
- 使用 Markdown 格式
- 🔴 🟡 🔵 分类图标保留
- 每条可操作项附带 → 建议操作
- 无可操作项时不推送
```

### 提示词适配

将 OpenClaw 分拣阶段的提示词适配为中文：

```text
读取 /tmp/openclaw/email-monitor-pending.json。
按照 EMAIL_MONITOR_PREFS.md 规则分拣邮件。
仅对需要我关注的邮件通过企业微信群机器人发送提醒。
使用表情分类，为可操作项添加 → 建议操作行。
每条提醒之间空一行。
如果没有可操作的邮件，跳过本次提醒。
```

### 实用建议

- **IMAP 方案是最通用的选择**：腾讯、网易、阿里企业邮箱均支持标准 IMAP 协议，一套预取脚本稍作修改即可适配多个邮箱系统
- **时区配置**：将预取脚本中的时区改为 `Asia/Shanghai`（CST +8），避免因时区差异遗漏或重复拉取邮件
- **中文邮件头解码**：国内邮件的主题和发件人经常使用 MIME 编码（如 `=?UTF-8?B?...?=`），预取脚本中需加入解码逻辑（上方示例已包含）
- **回复链标记差异**：国内邮件的回复链标记除了 `From:`、`-----Original` 外，还可能出现 `发件人：`、`原始邮件` 等中文标记，建议在 `strip_reply_chains` 中补充

> **安全提醒**：所有邮箱凭证（授权码、API 密钥、Webhook 地址）必须通过环境变量传递，绝不硬编码在脚本或配置文件中。

### 相关链接

- [腾讯企业邮箱开放接口](https://www.qqbizmail.com/api/) — 腾讯企业邮箱 API 文档
- [网易企业邮箱接口文档](https://qiye.163.com/help/l-24.html) — 网易企业邮箱 API 文档
- [Azure 中国门户](https://portal.azure.cn/) — 世纪互联版 M365 应用注册
- [Azure 中国 Graph API 文档](https://docs.azure.cn/) — 世纪互联版 Microsoft Graph 文档
- [mcporter 技能](https://playbooks.com/skills/openclaw/skills/mcporter) — Microsoft 365 MCP 集成

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/46)
