# AI 视频剪辑

> 含国内适配：剪映 / 必剪 / 竖版裁切 / 抖音·小红书·视频号分发

剪视频意味着打开时间线编辑器，拖拽片段，一个接一个地点菜单。裁头尾、加字幕、批量调色——这些重复操作每次都要手动完成，积少成多能吃掉一整天。

这个用例把视频剪辑变成一场对话：描述你要什么改动，丢进文件，拿回成品。不需要时间线，不需要 GUI（图形界面）。

> **与"短视频矩阵自动化"的区别**：PR #45 的短视频矩阵用例覆盖从选题到多平台分发的全链路。本用例专注**剪辑操作本身**——裁剪、合并、字幕、调色、竖版裁切。两者互补：你可以用本用例完成精细剪辑，再用矩阵用例完成分发。

## 它能做什么

- **裁剪与合并** — 用自然语言描述时间戳，自动裁切片段、合并多个素材
- **背景音乐** — 添加 BGM 并自动闪避人声（audio ducking，即音乐自动压低让人声突出）
- **字幕生成与烧录** — 语音识别生成字幕，支持 50+ 语言，可直接烧录到画面或导出 SRT（SubRip 字幕格式）文件
- **调色** — 用描述性语言调色（"暖一些"、"匹配第一个片段的色调"），无需手调参数
- **竖版裁切** — 一键将横版素材裁切为 9:16 竖版，适配抖音 / Reels / Shorts
- **批量处理** — 对多个文件执行相同编辑指令

## 所需技能

- [video-editor-ai](https://clawhub.ai/imo14reifey/video-editor-ai) — 基于 NemoVideo 的对话式视频编辑技能，支持裁剪、BGM、字幕、调色和导出。技能源码开源在 [nemovideo/nemovideo_skills](https://github.com/nemovideo/nemovideo_skills)（SKILL.md 约 450 行，包含路由表和状态差分逻辑）
- [ai-subtitle-generator](https://clawhub.ai/skills/ai-subtitle-generator) — 自动字幕生成、烧录和 SRT 导出

> **成熟度说明**：video-editor-ai 技能的底层仓库 `nemovideo_skills` 目前 star 数较少（< 50），但已在 ClawHub 官方注册表上线，且有独立社区实践者验证（见"相关链接"中的 DEV Community 文章）。建议在非关键素材上先行测试。

## 如何设置

1. 安装技能：

```bash
clawhub install video-editor-ai
clawhub install ai-subtitle-generator
```

首次运行会自动获取 100 次免费额度，无需注册账号。系统会在 `~/.config/nemovideo/client_id` 保存持久化客户端标识，避免匿名令牌的频率限制（每客户端每 7 天一个令牌）。

2. 拖入视频文件，描述你的编辑意图：

```text
Trim this video from 0:15 to 1:30, add background music (something upbeat),
and burn subtitles in English.
```

> 中文说明：从 0:15 裁到 1:30，加一段节奏轻快的背景音乐，烧录英文字幕。

3. 批量处理——描述规则，智能体逐个执行：

```text
I have 5 clips in /videos/raw/. For each one:
- Crop to 9:16 vertical
- Add auto-generated captions at the bottom
- Export as mp4 at 1080p
```

> 中文说明：把 /videos/raw/ 下的 5 个片段逐个裁切为 9:16 竖版，底部加自动字幕，导出为 1080p 的 mp4。

4. 调色——用自然语言描述目标风格：

```text
Color grade this clip to match a warm sunset look.
Boost the orange tones slightly and reduce blue in the shadows.
```

> 中文说明：给这个片段调出暖色日落风格，稍微加强橙色调，压暗部的蓝色。

智能体负责 API 调用、轮询渲染进度、交付最终文件。长时间操作（视频生成通常需要 100–300 秒）时会每 2 分钟显示一次进度提示。

## 实用建议

- **精确描述时间戳和输出格式**：说清楚"导出为 mp4，1080p"比模糊的"高清"更可靠
- **字幕指定源语言**：如果原始音频不是英语，明确告知语言（如"源语言是普通话"），识别准确率会显著提高
- **调色用描述而非数值**：说"暖色日落调"比"色温 +500K"效果更好——智能体会根据描述选择最合适的参数
- **先用非关键素材测试**：新技能建议先拿测试片段跑通全流程，确认效果再处理正式素材
- **隐私注意**：避免上传包含人脸、证件、机密画面的素材，除非你已确认服务商的数据留存和隐私政策

## 中国用户适配

### 国内工具对比

| 工具 | 优势 | 局限 | 与 OpenClaw 的关系 |
|------|------|------|-------------------|
| 剪映（CapCut） | 免费、功能全面、AI 字幕/调色 | **无官方 API**，自动化依赖第三方 UI 模拟（[JianYingApi](https://github.com/JianYing-Automation/JianYingApi)，240+ star），可能因版本更新失效 | OpenClaw 通过 API 直接控制，无需模拟 GUI |
| 必剪（B站出品） | 免费、集成 AI 数字分身和弹幕字幕 | 功能偏向 B 站生态，无 API 接口 | OpenClaw 更适合跨平台批量处理 |
| 快影（快手出品） | AI 智能剪辑、AI 文案成片 | 无 API，仅移动端 | OpenClaw 支持桌面端自动化 |

> **核心差异**：国内工具强在 GUI 易用性和免费 AI 功能，但都缺少可编程接口。OpenClaw + video-editor-ai 的优势在于**可脚本化、可批量化、可集成到更大的自动化流水线中**。

### 中文字幕优化

video-editor-ai 底层使用 Whisper 级语音识别，中文普通话识别率较好，但对方言和专业术语可能不准确。建议：

- 在提示词中明确指定 `source language: Mandarin Chinese`
- 如需更高准确率，可先用讯飞听见或飞书妙记生成 SRT 文件，再让智能体烧录：

```text
Burn the subtitles from /videos/captions.srt onto this video.
Position them at the bottom center, white text with black outline.
```

> 中文说明：把 /videos/captions.srt 字幕烧录到视频上，居中底部，白字黑边。

### 竖版裁切（抖音 / 小红书 / 视频号）

国内短视频平台均以 9:16 竖版为主。批量裁切横版素材为竖版是高频需求：

```text
I have 10 horizontal videos in /videos/landscape/.
For each one:
- Smart crop to 9:16, keep the speaker centered
- Add Chinese captions at the bottom (source language: Mandarin Chinese)
- Export as mp4 at 1080p
- Save to /videos/vertical/
```

> 中文说明：把 10 个横版视频智能裁切为 9:16 竖版，保持说话人居中，底部加中文字幕，导出 1080p mp4 到 /videos/vertical/。

### 与国内分发工具搭配

剪辑完成后，可以用开源工具 [social-auto-upload](https://github.com/dreammis/social-auto-upload)（9,400+ star）自动发布到抖音、小红书、视频号、B 站等平台。该工具基于 Playwright 浏览器自动化，支持定时发布和批量上传。

> **平台风控提醒**：各短视频平台对自动化发布有检测机制。建议控制发布频率、避免同一素材原封不动发多平台、每次发布间隔至少 5–10 分钟。使用自动化工具发布视频须遵守各平台社区规范。

## 相关链接

- [video-editor-ai 技能源码](https://github.com/nemovideo/nemovideo_skills) — SKILL.md 开源仓库
- [NemoVideo + OpenClaw 工作流](https://www.nemovideo.com/blog/openclaw-nemovideo-workflow-2026) — 官方集成教程
- [How I Built an AI Video Editor as an OpenClaw Skill](https://dev.to/weizhang_dev/how-i-built-an-ai-video-editor-as-an-openclaw-skill-103j) — 社区实践者开发记录
- [JianYingApi](https://github.com/JianYing-Automation/JianYingApi) — 第三方剪映自动化接口（240+ star）
- [social-auto-upload](https://github.com/dreammis/social-auto-upload) — 多平台视频自动发布工具（9,400+ star）
- [Whisper (OpenAI)](https://github.com/openai/whisper) — 本地语音转文字

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/ai-video-editing.md)
