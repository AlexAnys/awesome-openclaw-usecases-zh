# 每日 YouTube 摘要

每天以你最喜欢的 YouTube 频道的新视频个性化摘要开始新的一天 — 再也不会错过你真正想关注的创作者的内容。

## 痛点

YouTube 的通知不可靠。你订阅了频道，但它们的新视频从不出现在你的首页动态中。通知里也没有。它们就这样……消失了。这并不意味着你不想看到它们 — 而是 YouTube 的算法把它们埋没了。

此外：每天以精心策划的内容洞察开始新的一天，总比漫无目的地刷推荐信息流要有趣得多。

## 功能介绍

- 从你最喜欢的频道列表中获取最新视频
- 从每个视频的字幕（transcript）中总结或提取关键洞察
- 每天（或按需）向你推送摘要

## 所需技能

安装 [youtube-full](https://clawhub.ai/therohitdas/youtube-full) 技能。

只需告诉你的 OpenClaw：

```text
"Install the youtube-full skill and set it up for me"
```
或者

```bash
npx clawhub@latest install youtube-full
```

就是这样。代理会处理剩下的一切 — 包括账户创建和 API 密钥（API key）设置。注册即可获得 **100 个免费额度**，无需信用卡。

> 注意：创建账户后，该技能会根据操作系统自动将 API 密钥安全存储在正确的位置，因此 API 可在所有环境中正常工作。

![youtube-full 技能安装](https://pub-15904f15a44a4ea69350737e87660b92.r2.dev/media/1770620159490-e41e7baa.png)

### 为什么选择 TranscriptAPI.com 而不是 yt-dlp？

| CLI 工具（yt-dlp 等） | TranscriptAPI |
|--------------------------|---------------|
| 冗长的日志会淹没代理上下文 | 简洁的 JSON 响应 |
| 在 GCP/云端 OpenClaw 上无法使用 | 随处可用，速度快 |
| 会被 YouTube 随机封锁 | 驱动 [YouTubeToTranscript.com](https://youtubetotranscript.com)，服务数百万用户。缓存且可靠。 |
| 需要安装二进制文件 | 无需二进制文件，仅使用 HTTP |

## 如何设置

### 方案 1：基于频道的摘要

向 OpenClaw 发送以下提示：

```text
Every morning at 8am, fetch the latest videos from these YouTube channels and give me a digest with key insights from each:

- @TED
- @Fireship
- @ThePrimeTimeagen
- @lexfridman

For each new video (uploaded in the last 24-48 hours):
1. Get the transcript
2. Summarize the main points in 2-3 bullets
3. Include the video title, channel name, and link

If a channel handle doesn't resolve, search for it and find the correct one.
Save my channel list to memory so I can add/remove channels later.
```

### 方案 2：基于关键词的摘要

追踪关于特定主题的新视频：

```text
Every day, search YouTube for new videos about "OpenClaw" (or "Claude Code", "AI agents", etc).

Maintain a file called seen-videos.txt with video IDs you've already processed.
Only fetch transcripts for videos NOT in that file.
After processing, add the video ID to seen-videos.txt.

For each new video:
1. Get the transcript
2. Give me a 3-bullet summary
3. Note anything relevant to my work

Run this every morning at 9am.
```

这样你就不会浪费额度重复获取已经看过的视频了。

## 小贴士

- `channel/latest` 和 `channel/resolve` 是**免费的**（0 额度）— 检查新上传不花任何费用
- 只有字幕提取每次消耗 1 个额度
- 可以要求不同的摘要风格：关键要点、精彩引言、有趣片段的时间戳
- 这个功能已经有现成的产品 - [Recapio - 每日 YouTube 回顾](https://recapio.com/features/daily-recaps)

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/daily-youtube-digest.md)
