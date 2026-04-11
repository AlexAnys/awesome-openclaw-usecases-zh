# 数字人格蒸馏（Digital Persona Distillation）

> 含国内适配：微信 / 飞书 / 钉钉聊天记录采集 + 隐私与伦理专项提醒

用 [immortal-skill](https://github.com/agenmod/immortal-skill) 从散落在 12+ 平台的聊天记录和文档中，四维结构化蒸馏出一个人的数字分身（Digital Persona）——不是把聊天记录塞进向量库，而是提取 TA 怎么做事、怎么说话、经历过什么、是什么人，生成符合 OpenClaw Skill 标准的可加载人格档案。

## 功能介绍

- **四维蒸馏**：从聊天记录中提取程序性知识（Procedure）、互动风格（Interaction）、记忆经历（Memory）、性格价值观（Personality）四个维度，而非简单的关键词或嵌入向量
- **12+ 平台采集**：飞书、钉钉、微信、iMessage、Telegram、WhatsApp、Slack、Discord、Email、Twitter/X、Google Takeout、手动文件导入，每个平台有独立的 Python 采集器
- **7 种角色模板**：自己、同事、导师、亲人、伴侣/前任、朋友、公众人物——不同角色的蒸馏侧重和伦理要求不同
- **证据分级**：每条提取结果标注 `verbatim`（原话）、`artifact`（文档留痕）或 `impression`（他人印象），矛盾之处不强行统一，单独记录在 `conflicts.md`
- **输出即 Skill**：蒸馏产物是一个标准的 OpenClaw Skill 目录（SKILL.md + 维度文件 + manifest.json），AI 加载后直接可用
- **版本管理**：支持快照、回滚、增量追加材料，人格档案可持续进化

## 痛点

你和某个重要的人——可能是离职的同事、退休的导师、远去的亲人——有过大量对话，但这些对话散落在微信、飞书、Telegram、iMessage 等十几个 App 里。你想保留的不是聊天记录本身，而是 TA 怎么思考问题、怎么做决定、怎么跟你说话的那种"感觉"。

直接把聊天记录扔进 RAG 向量库，搜出来的是碎片化的消息，不是一个完整的人。你需要的是一个能理解"TA 遇到这种事会怎么反应"的数字分身，而不是一个搜索引擎。

这也是一个"先蒸自己"的工具——在 AI 时代，你的数字人格迟早会被某种形式蒸馏，与其被动等别人来定义你，不如先把自己蒸明白。

## 所需技能

- [immortal-skill](https://github.com/agenmod/immortal-skill)（SKILL.md）— 主蒸馏引擎，包含采集器、提取 Prompt、合并策略和 CLI 工具

可选扩展（同仓库内）：

- [distill-shield-skill](https://github.com/agenmod/immortal-skill/tree/main/distill-shield-skill) — 防蒸馏保护（身份编码 + 蒸馏许可 + 保护锁三层纵深）
- [distill-protocol-skill](https://github.com/agenmod/immortal-skill/tree/main/distill-protocol-skill) — 蒸馏协议，声明蒸馏产物的使用边界
- [steamer-skill](https://github.com/agenmod/immortal-skill/tree/main/steamer-skill) — 蒸笼，从公众人物的公开言论中蒸馏认知框架

## 如何设置

### 1. 获取 immortal-skill

克隆仓库并确保 Python 3.9+ 可用：

```bash
git clone https://github.com/agenmod/immortal-skill.git
cd immortal-skill
```

### 2. 配置数据平台凭证

根据你的数据来源，配置对应平台的凭证。以飞书为例：

```bash
# 交互式配置飞书凭证（需要飞书自建应用的 App ID / App Secret）
python3 kit/immortal_cli.py setup feishu
```

也可以通过环境变量配置：

```bash
export FEISHU_APP_ID="your_app_id"
export FEISHU_APP_SECRET="your_app_secret"
```

> 各平台的详细配置步骤见仓库内的 [PLATFORM-GUIDE.md](https://github.com/agenmod/immortal-skill/blob/main/docs/PLATFORM-GUIDE.md)，包含保姆级数据获取指南。

### 3. 在 OpenClaw 中加载 Skill

将 `immortal-skill` 仓库作为 Skill 目录加载：

```json
{
  "skills": {
    "immortal": {
      "path": "/path/to/immortal-skill"
    }
  }
}
```

或者直接在对话中告诉 AI：

```
工作目录设为 immortal-skill 仓库根。请先读 FOR_AI.md，再按我的意图执行。
```

### 4. 开始蒸馏

**方式一：自然语言触发**

直接在 OpenClaw 对话中说：

```
帮我把李工蒸馏成 Skill，他的飞书记录在这儿。
```

```
我想蒸馏我自己，微信 + Twitter 全上。
```

```
蒸馏我奶奶，微信记录我导出来了。
```

AI 会按照 SKILL.md 中的 Phase 0-7 流程自动执行：选择角色模板 -> 伦理确认 -> 收集材料 -> 分维度提取 -> 合并冲突 -> 初始化目录 -> 封包登记 -> 告知用户。

**方式二：CLI 手动挡**

```bash
# 查看支持的平台
python3 kit/immortal_cli.py platforms

# 扫描飞书群聊列表
python3 kit/immortal_cli.py collect --platform feishu --scan

# 从微信数据库采集指定联系人的聊天记录
python3 kit/immortal_cli.py collect --platform wechat --db ~/wechat.db --channel "张三"

# 从 iMessage 采集
python3 kit/immortal_cli.py collect --platform imessage --scan

# 手动导入已有文件
python3 kit/immortal_cli.py import ~/chat-export.txt --output corpus/chat.md

# 初始化人格目录
python3 kit/immortal_cli.py init --slug grandma --persona family

# 封包登记
python3 kit/immortal_cli.py stamp --slug grandma --sources "wechat:chat,paste:notes"

# 创建版本快照
python3 kit/immortal_cli.py snapshot --slug grandma --note "第一版"
```

### 5. 蒸馏产物结构

一个人 = 一个目录 = 一个可加载的 OpenClaw Skill：

```
skills/immortals/grandma/
├── SKILL.md          ← AI 读这个就知道"奶奶是谁"
├── interaction.md    ← 怎么说话、怎么唠叨、怎么关心人
├── memory.md         ← 讲过什么故事、经历过什么
├── personality.md    ← 是什么样的人、在乎什么
├── procedure.md      ← 怎么做事（适用于同事/导师等角色）
├── conflicts.md      ← 不同来源说法不一致的地方
└── manifest.json     ← 元数据（来源、时间、指纹、证据覆盖度）
```

把这个目录放进 OpenClaw 的 skills 路径，AI 就能用该人格的语气和思维方式与你对话。

## 真实案例

**蒸馏自己：**

```
你：帮我蒸馏自己，微信和 Twitter 的记录都用上。
```

AI 按流程执行，约 2-5 分钟后返回：

> **蒸馏完成 — 陈韵**
>
> **证据覆盖度：** procedure=5v+3a; interaction=4v+2a+2i; memory=3v+1a; personality=2v+4a+3i
>
> **互动风格摘要：**
> - 文字偏长，喜欢用类比解释概念（`artifact`）
> - 常用"你可以理解为...""换句话说..."引导对方理解（`verbatim`）
> - 面对不合理需求时先问"用户的真实诉求是什么"而非直接拒绝（`verbatim`）
>
> **性格摘要：**
> - 产品设计偏好极简，用户心智模型驱动
> - 对审美有洁癖，不接受"照着竞品抄"的指令（`verbatim`："抄是最没有创意的事"）
>
> **局限性：** 数据主要来自微信聊天和 Twitter 帖子，职场文档覆盖不足。部分 personality 条目来自朋友评价（已标注 `impression`）。
>
> 产物路径：`./skills/immortals/chen-yun-demo/`

**蒸馏跑路的同事：**

```
你：李工跑路了，帮我蒸馏他的工作方式。飞书群聊"产品技术群"里有他的消息。
```

> AI 选择"同事"角色模板，仅提取 procedure（工作流程）和 interaction（沟通风格）两个维度，跳过 memory 和 personality 中与工作无关的部分。

**蒸馏公众人物（蒸笼模式）：**

```
你：蒸馏一个巴菲特，我要他的投资决策框架当参谋。
```

> AI 使用蒸笼（steamer-skill），从公开演讲、致股东信、访谈视频等公开资料中提取认知框架，生成可对话的"投资参谋"Skill。

## 隐私与伦理提醒

这是一个涉及个人隐私的敏感工具。immortal-skill 内置了伦理检查流程（Phase 1），但使用者必须自行承担最终责任：

### 必须遵守的底线

- **蒸馏他人前必须获得知情同意**：除了自己和公众人物的公开资料外，蒸馏任何人都应获得本人（或家属）的明确同意
- **聊天记录中的第三方信息必须脱敏**：你和张三的聊天里可能提到了李四——李四的信息不应出现在张三的人格档案中
- **蒸馏产物不得用于冒充、骚扰或欺骗**：数字分身是辅助回忆和知识传承的工具，不是deepfake
- **公众人物蒸馏仅限公开资料**：公开演讲、已出版著作、公开访谈可以用；私人对话、泄露内容不可以

### 工具内置的保护机制

| 机制 | 说明 |
|------|------|
| **角色级伦理模板** | 不同角色（同事/亲人/前任/公众人物）有不同的伦理检查清单，在蒸馏开始前强制确认 |
| **证据分级标注** | 每条提取结果标注来源类型（verbatim/artifact/impression），让用户和 AI 都能判断可信度 |
| **冲突不强行统一** | 矛盾的信息不会被"合理化"，而是如实记录在 conflicts.md 中 |
| **防蒸馏扩展** | distill-shield-skill 提供三层纵深防御（身份编码/蒸馏许可/保护锁），帮助你保护自己不被他人未授权蒸馏 |
| **蒸馏协议扩展** | distill-protocol-skill 帮你声明蒸馏产物的使用边界——能不能蒸、能不能商用、数字分身能不能替代你工作 |

### 法律风险提示

- 中国《个人信息保护法》（PIPL）对个人信息处理有严格要求，聊天记录属于个人信息
- 蒸馏他人的聊天记录可能涉及个人信息处理的合法性问题，建议咨询法律专业人士
- 蒸馏产物如用于商业用途，可能涉及肖像权、名誉权等人格权问题

## 实用建议

- **先蒸自己再蒸别人**：用自己的数据跑通全流程，理解产物质量和局限性，再考虑蒸馏他人。
- **数据量决定质量**：单一平台几百条消息只能得到粗糙的印象；跨平台几千条消息才能提取出有层次的人格档案。
- **证据覆盖度是质量指标**：`verbatim`（原话）占比越高，蒸馏结果越可靠。如果大部分条目都是 `impression`，说明需要补充更多一手材料。
- **不要追求完美**：人本来就是矛盾的，conflicts.md 里记录的矛盾不是 bug，是 feature。
- **定期增量更新**：人是会变的。用版本管理功能定期追加新材料，让数字分身跟着真人一起进化。
- **微信数据获取是难点**：微信没有官方导出 API，需要借助 [WeChatMsg](https://github.com/LC044/WeChatMsg)（Windows）或 [WechatExporter](https://github.com/BlueMatthew/WechatExporter)（iOS）等第三方工具先导出，再用 immortal-skill 的微信采集器解析。

## 中国用户适配

### 国内平台采集

immortal-skill 原生支持国内主流平台，无需额外适配：

| 平台 | 采集方式 | 说明 |
|------|---------|------|
| **飞书** | API 自动拉取 | 需飞书自建应用的 App ID / App Secret |
| **钉钉** | API 自动拉取 | 需钉钉企业内部应用凭证 |
| **微信** | 本地数据库解析 | 需先用第三方工具导出 SQLite 数据库 |
| **手动导入** | 任意文件 | 支持 TXT / JSON / CSV / Markdown |

### 微信数据获取路径

微信是大多数中国用户最核心的聊天平台，但数据获取门槛最高：

```
推荐路径：
  Windows 用户 → WeChatMsg (github.com/LC044/WeChatMsg) 导出 CSV → immortal-skill 解析
  iOS 用户 → iTunes 备份 → WechatExporter 导出 → immortal-skill 解析
  macOS 用户 → 微信 Mac 版本地数据库 → immortal-skill 直接读取
```

### 蒸馏产物与飞书/钉钉集成

蒸馏完成后，可以将产物 Skill 接入飞书或钉钉的 AI 助手：

```text
我已经蒸馏好了李工的工作方式 Skill，请加载 skills/immortals/li-gong/ 目录。
以后当我问"李工会怎么处理这种情况"时，用他的思维方式回答。
```

适用场景：新人入职时加载离职同事的"工作方式 Skill"，在飞书群里随时查询"这件事李工以前是怎么做的"。

## 与"第二大脑"和"知识库 RAG"的区别

| 维度 | 第二大脑 / 知识库 RAG | 数字人格蒸馏 |
|------|----------------------|------------|
| 存什么 | 知识片段、笔记、文章 | 一个人的思维方式和行为模式 |
| 搜索方式 | 语义搜索，返回相关段落 | 人格加载，AI 以该人的方式回应 |
| 输出 | 信息检索结果 | 角色扮演式对话 |
| 数据粒度 | 文档级别 | 对话级别（逐条消息提取） |
| 证据追溯 | 通常有 | 有，且分三级（原话/文档/印象） |
| 适用场景 | "关于 X 我之前记了什么" | "遇到这种事，TA 会怎么想" |

## 相关链接

- [immortal-skill 仓库](https://github.com/agenmod/immortal-skill) — 主蒸馏引擎（MIT 协议）
- [人格广场（在线体验）](https://agenworld.com/market) — 预制公众人物人格蒸馏结果，可直接对话
- [WeChatMsg](https://github.com/LC044/WeChatMsg) — Windows 微信聊天记录导出工具
- [WechatExporter](https://github.com/BlueMatthew/WechatExporter) — iOS 微信聊天记录导出工具

---

**原始项目**：[immortal-skill](https://github.com/agenmod/immortal-skill)（491 stars，MIT 协议）
