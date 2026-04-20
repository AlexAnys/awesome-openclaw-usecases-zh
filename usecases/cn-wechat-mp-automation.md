# 微信公众号自动发布

公众号运营者的日常：追热点 → 定选题 → 写文章 → Markdown 转排版 → 上传封面图 → 复制粘贴到编辑器 → 预览 → 发布。每篇推文从构思到上线至少两小时，遇到排版微调更是反复折腾。

这个用例通过 OpenClaw Skill 打通"AI 写稿 → Markdown 排版转换 → 自动上传草稿箱"的链路，把重复劳动压缩到一句话指令。

## 它能做什么

- **热点选题**：聚合微博、抖音、B 站等平台热榜，AI 从运营视角推荐适合公众号的话题
- **文章撰写**：根据选题生成符合公众号阅读习惯的长文，包括标题、正文、摘要
- **Markdown 转排版**：将 Markdown 转为微信编辑器兼容的富文本，支持多种主题样式和代码高亮
- **素材上传**：自动上传封面图和正文配图到微信素材库
- **草稿箱推送**：一键将排版后的文章推送到公众号草稿箱，返回 media_id（素材标识）
- **发布管理**：对于有权限的账号，可通过 API 提交草稿发布并轮询发布状态

> **关键限制**：大多数个人订阅号只能自动推送到草稿箱，最终发布仍需在公众号后台或"公众号助手"App 手动点击。详见下方"准入要求"章节。

## 所需技能

社区已有多个可用的 Skill，按成熟度排列：

| Skill | 功能 | 成熟度 | 安装方式 |
|-------|------|--------|---------|
| [md2wechat-skill](https://github.com/geekjourneyx/md2wechat-skill) | Markdown 转排版 + 草稿箱推送，支持多主题、AI 去痕、批量发布 | 🟢 1,400+ stars，活跃 | `brew install geekjourneyx/tap/md2wechat` |
| [wechat-publisher](https://github.com/0731coderlee-sudo/wechat-publisher) | 一键 Markdown 到草稿箱 | 🟡 社区小项目，自行评估 | `npx skills add 0731coderlee-sudo/wechat-publisher` |
| [wechat-publisher](https://github.com/lucianaib0318/wechat-publisher)（lucianaib0318 版） | 热榜驱动的公众号发布助手，与下方选题工具配套 | 🟡 社区小项目，自行评估 | `npx skills add lucianaib0318/wechat-publisher` |
| [china-hot-ranks](https://github.com/lucianaib0318/china-hot-ranks) | 全网热榜聚合（微博/抖音/B 站/百度等） | 🟡 社区小项目，自行评估 | `npx skills add lucianaib0318/china-hot-ranks` |
| [wechat-topic-selector](https://github.com/lucianaib0318/wechat-topic-selector) | 公众号选题推荐 | 🟡 社区小项目，自行评估 | `npx skills add lucianaib0318/wechat-topic-selector` |

需要 Python 3.10+（部分 Skill 依赖）和 Node.js 18+（OpenClaw 运行环境）。

## 准入要求与权限说明

微信公众号 API 的权限因账号类型不同而差异极大，**在开始之前必须确认你的账号类型**：

| 能力 | 个人订阅号 | 企业订阅号（已认证） | 企业服务号（已认证） |
|------|-----------|-------------------|-------------------|
| 获取 access_token（访问令牌） | ✅ | ✅ | ✅ |
| 素材管理（上传图片/缩略图） | ✅ | ✅ | ✅ |
| 草稿箱接口（新建/修改/查询草稿） | ✅ | ✅ | ✅ |
| 发布接口 freepublish（提交草稿发布） | ⚠️ 已回收 | ✅ | ✅ |
| 数据统计接口 | ❌ | ✅ | ✅ |
| 模板消息 / 微信支付 | ❌ | ❌ | ✅ |

> **⚠️ 重要变更**：自 2025 年 7 月起，微信官方已回收个人主体账号和未认证企业账号的 `freepublish`（发布）接口权限。这意味着个人订阅号可以通过 API 创建草稿，但无法通过 API 直接发布，最终发布必须手动完成。

**获取 API 凭证**：登录 [微信公众平台](https://mp.weixin.qq.com/) → 设置与开发 → 基本配置 → 获取 AppID（应用标识）和 AppSecret（应用密钥）。

## 如何设置

### 方案一：md2wechat-skill（推荐，功能最全）

1. 安装工具：
```bash
# macOS
brew install geekjourneyx/tap/md2wechat

# 或通用安装脚本
curl -fsSL https://github.com/geekjourneyx/md2wechat-skill/releases/download/v2.0.5/install.sh | bash
```

2. 初始化配置（交互式填入 AppID 和 AppSecret）：
```bash
md2wechat config init
```

3. 在微信公众平台后台添加服务器 IP 白名单：
```bash
# 查询你的公网 IP
curl ifconfig.me
```
将输出的 IP 地址添加到：微信公众平台 → 设置与开发 → 基本配置 → IP 白名单。

4. 在 OpenClaw 对话中使用：
```text
把下面这篇 Markdown 转成公众号排版并推送到草稿箱：

# 标题：为什么程序员应该学会写作
（正文内容...）
```

### 方案二：wechat-publisher Skill

1. 安装 Skill：
```bash
npx skills add 0731coderlee-sudo/wechat-publisher
```

2. 在 OpenClaw 的 TOOLS.md 中配置凭证（使用环境变量，不要硬编码）：
```text
## wechat-publisher 配置

环境变量：
- WECHAT_APP_ID：你的公众号 AppID
- WECHAT_APP_SECRET：你的公众号 AppSecret
```

3. 配置 IP 白名单（同方案一步骤 3）。

4. 发布流程：
```text
帮我写一篇关于"AI 编程工具对比"的公众号文章，写完后推送到草稿箱。

用热辣风格重写这篇技术文章，配上吸引眼球的标题，然后发到公众号草稿箱。
```

### 方案三：全自动选题 + 撰写 + 发布流水线

组合安装 lucianaib0318 的三个配套 Skill 实现端到端自动化（注意：这里的 wechat-publisher 是 [lucianaib0318 版](https://github.com/lucianaib0318/wechat-publisher)，与方案二的 [0731coderlee-sudo 版](https://github.com/0731coderlee-sudo/wechat-publisher) 是不同项目）：

```bash
npx skills add lucianaib0318/china-hot-ranks
npx skills add lucianaib0318/wechat-topic-selector
npx skills add lucianaib0318/wechat-publisher
```

一句话触发完整流程：
```text
看一下今天的全网热榜，帮我挑一个适合我的公众号的技术话题，写一篇 2000 字左右的深度文章，排版后推送到草稿箱。
```

OpenClaw 会依次调用热榜聚合 → 选题推荐 → 文章生成 → 排版转换 → 草稿箱推送。

## 实用建议

- **先跑通"草稿箱推送"再做其他**：确认 AppID/AppSecret 和 IP 白名单配置正确，能成功推送一篇测试文章到草稿箱，再扩展选题和批量功能
- **人工审核不可省略**：AI 生成的文章必须检查事实准确性、价值观合规性和品牌调性，尤其是涉及数据、法规、医疗健康等敏感话题
- **图片使用公网可访问 URL**：正文配图需要先上传到微信素材库才能在文章中引用，本地图片路径无法直接使用
- **标题控制在 30 字以内**：微信公众号标题过长会被截断，影响打开率
- **善用"公众号助手"App 做最后一步**：个人订阅号无法通过 API 发布，但可以在手机上用"公众号助手"App 从草稿箱一键发布，体验接近全自动

### 与小红书自动化的差异

| 维度 | 微信公众号 | 小红书 |
|------|-----------|--------|
| 内容形式 | 长文为主（1000-5000 字） | 图文笔记为主（短文案 + 封面图） |
| 自动化方式 | 官方 API（草稿箱/发布接口） | RPA 浏览器自动化（无官方 API） |
| 发布限制 | 订阅号每日 1 条，服务号每月 4 条 | 无官方频率限制，但风控严格 |
| 账号门槛 | 需 AppID/AppSecret，部分接口需认证 | 仅需 Cookie 登录 |
| 封号风险 | API 调用合规风险低 | RPA 操作有指纹检测风险 |

> **⚠️ 风控提醒**
>
> - **内容审核**：微信对公众号文章有严格的内容审核机制，涉及政治敏感、虚假信息、违规营销等内容会被拦截甚至封号。AI 生成内容尤其需要人工把关。
> - **发文频率限制**：订阅号每天只能群发 1 条消息（可包含多篇文章），服务号每月只能群发 4 条。通过"发布"接口发布的文章不占群发额度，但仍受审核约束。
> - **API 调用限制**：每个接口有每日调用上限（具体数值因账号类型和接口而异），超限会返回错误码 45009。每月有 10 次清零机会。
> - **凭证安全**：AppSecret 泄露等同于账号被接管。务必通过环境变量传递，不要写入代码或提交到版本控制系统。
> - **IP 白名单**：只有白名单内的 IP 才能调用 API。如果使用云服务器，注意 IP 变更后需及时更新白名单。

## 相关链接

- [微信公众号开发者文档 - 订阅号 API](https://developers.weixin.qq.com/doc/subscription/api/)
- [微信公众号开发者文档 - 发布接口](https://developers.weixin.qq.com/doc/subscription/api/public/api_freepublish_submit)
- [md2wechat-skill - GitHub](https://github.com/geekjourneyx/md2wechat-skill)（1,400+ stars，活跃维护）
- [wechat-publisher（0731coderlee-sudo 版）- GitHub](https://github.com/0731coderlee-sudo/wechat-publisher)
- [wechat-publisher（lucianaib0318 版）- GitHub](https://github.com/lucianaib0318/wechat-publisher)
- [wechatpy - Python 微信 SDK](https://github.com/wechatpy/wechatpy)（4,276 stars）
- [腾讯云 - 手把手教你用 OpenClaw + Skill 实现微信公众号全自动创作发布](https://cloud.tencent.com/developer/article/2640353)
- [博客园 - 使用 OpenClaw + Skill 自动发布微信公众号文章](https://www.cnblogs.com/xuxueli/p/19721838)
