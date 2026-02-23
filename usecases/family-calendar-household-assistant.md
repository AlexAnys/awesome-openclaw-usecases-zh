# 家庭日历聚合与家务助手

现代家庭需要同时管理五个甚至更多日历——工作、个人、共享家庭、孩子学校、课外活动——分布在不同平台和格式中。重要事件会因为没有统一视图而被遗漏。与此同时，家务协调（购物清单、储藏室库存、预约安排）通过零散的短信进行，很容易被淹没。

这个用例将 OpenClaw 变成一个始终在线的家庭协调员：聚合日历生成早间简报、监控消息中的可操作事项，以及通过共享聊天界面管理家务后勤。

## 痛点

- **日历碎片化**：工作日历有安全限制，无法共享。学校日历以 PDF 或手写网页的形式送达。夏令营时间表在邮件里。每天早上手动检查每个日历是不可持续的——而且"跨日历复制事件确实管用，直到我忘了，然后就有事情漏掉了。"
- **家务协调开销**："家里还有多少牛奶？"需要亲自检查冰箱，然后检查地下室储藏间，然后回复短信。这样的事情每周购物时都要重复多次。
- **错过预约**：预约确认以短信形式到达，然后就搁在那里没人处理——没有日历事件，没有路程时间缓冲，没有提醒。

## 功能介绍

- **早间简报**：将所有家庭日历聚合成每日摘要，通过你偏好的渠道发送
- **环境消息监控**：监视 iMessage/短信对话，在检测到预约时自动创建日历事件（牙医确认、会面计划等）
- **路程时间缓冲**：在检测到的预约前后添加交通时间段
- **家庭库存管理**：维护储藏室/冰箱物品的实时库存清单，任何一方都可以随时随地查询
- **购物协调**：跨食谱去重食材，跟踪库存不足的物品，生成购物清单
- **照片输入**：拍一张学校日历或冷冻柜内容的照片，智能体会将其处理为结构化数据

## 所需技能

- 日历 API 访问（Google Calendar、通过 `ical` 访问 Apple Calendar）
- `imessage` 技能用于消息监控（仅限 macOS）
- Telegram 或 Slack 用于共享家庭聊天界面
- 文件系统访问用于库存跟踪
- 相机/照片处理用于实体日历的 OCR（光学字符识别）

## 设置方法

### 1. 日历聚合

配置 OpenClaw 从所有家庭日历源拉取数据：

以下提示词让智能体每天早上聚合所有日历源并生成简报：

```text
## Calendar Sources

On morning briefing (8:00 AM):

1. Fetch my Google Work Calendar (read-only OAuth)
2. Fetch shared Family Google Calendar
3. Fetch partner's calendar (shared view)
4. Check ~/Documents/school-calendars/ for any new PDFs → OCR and extract events
5. Check recent emails for calendar attachments or event invitations

Compile into a single briefing:
- Today's events (all calendars, color-coded by source)
- Upcoming 3-day lookahead for conflicts
- Any new events added since yesterday
- Weather context for outdoor events

Deliver via Telegram/Slack family channel.
```

### 2. 环境消息监控

这是关键差异化功能——智能体在后台被动监视，在识别到可操作内容时采取行动：

以下提示词配置智能体定期检查消息并自动创建日历事件：

```text
## Message Monitoring (HEARTBEAT.md)

Every 15 minutes:
1. Check new iMessages across all conversations
2. Detect appointment-like patterns:
   - "Your appointment is confirmed for..."
   - "Can we meet on [date] at [time]?"
   - "Practice moved to Saturday at 3pm"
3. When detected:
   - Create calendar event with details
   - Add 30-minute driving buffer before AND after
   - Send confirmation to family Telegram: "Created: Dentist appointment, Tue 2pm. Added drive time 1:30-2:00 and 3:00-3:30."
   - If relevant to partner, add invite
4. Detect promise/commitment patterns:
   - "I'll send that over by Friday"
   - "Let's do dinner next week"
   → Create calendar hold or reminder
```

### 3. 家庭库存管理

以下提示词配置智能体管理家庭库存，支持照片、文字和收据等多种输入方式：

```text
## Pantry Tracking

Maintain ~/household/inventory.json with:
- Item name, quantity, location (fridge/pantry/basement)
- Last updated timestamp
- Low-stock threshold

Update methods:
- Photo: User sends photo of fridge/pantry → vision model extracts items
- Text: "We're out of eggs" / "Bought 2 gallons of milk"
- Receipt: Photo of grocery receipt → update inventory

Query: Either partner can ask via Telegram:
- "Do we have butter?" → Check inventory, respond with location and quantity
- "What's running low?" → List items below threshold
- "Generate grocery list" → Compile low-stock items + any recipe ingredients needed
```

## 关键洞察

- **被动优于主动**：最大的突破在于智能体无需被要求就能行动。检测短信中的预约并自动创建带路程缓冲的日历事件——"我没有让它这样做。它就是知道这是我想要的。"
- **Mac Mini 是最佳载体**：这个用例特别适合在家用 Mac Mini 上运行——iMessage 集成、Apple Calendar 和始终在线的可用性
- **从只读开始**：先从日历读取和消息监控开始，然后再启用写入操作（创建事件、发送消息）
- **共享 Telegram 频道**：让双方都能看到智能体在做什么——建立信任并尽早发现错误
- **照片输入被低估了**：拍一张学校日历 PDF 或冷冻柜内容的照片比打字快得多——而且视觉模型处理得很好

## 灵感来源

这个用例结合了多个社区模式：

- **日历聚合**：由 HN 用户 `angiolillo` 在一次 [Hacker News 讨论](https://news.ycombinator.com/item?id=46872465)中描述，他详细说明了每天早上分别检查工作、个人、家庭和孩子学校日历的痛苦。
- **环境消息监控**：由 [Sparkry AI](https://sparkryai.substack.com/p/24-hours-with-openclaw-the-ai-setup) 记录——当妻子收到一条牙科预约短信时，OpenClaw 自动创建了带 30 分钟路程缓冲的日历事件，无需被要求。在 [OpenClaw Showcase](https://openclaw.ai/showcase) 上也得到了验证，`@theaaron` 称基于聊天的日历管理是"我体验过的最好的 LLM 用途之一"。
- **家务协调**：Brandon Wang 的 [Clawdbot "Linguini"](https://brandon.wang/2026/clawdbot) 运行在家用 Mac Mini 上——处理短信跟进、从照片创建日历事件、跟踪 Airbnb 价格、处理冷冻柜库存照片，以及通过 iMessage 和 Slack 协调家务后勤。
- **储藏室跟踪**：多位 HN 用户讨论了维护家庭库存的价值（和挑战），`dns_snek` 指出："我5秒钟前放东西的地方都会忘记……这对我来说确实是个大问题，因为我会让东西过期。"

## 相关链接

- [OpenClaw iMessage 技能](https://github.com/openclaw/openclaw)
- [Google Calendar API](https://developers.google.com/calendar)
- [Apple Calendar (EventKit)](https://developer.apple.com/documentation/eventkit)
- [OpenClaw Showcase — 日历相关评价](https://openclaw.ai/showcase)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/family-calendar-household-assistant.md)
