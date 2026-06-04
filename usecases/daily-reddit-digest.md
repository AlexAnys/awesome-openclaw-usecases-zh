# 每日 Reddit 摘要
每天运行一次每日摘要，为你提供最喜欢的子版块中表现最好的帖子。

适用场景：

• 浏览子版块（热门/最新/置顶帖子）
• 按主题搜索帖子
• 拉取评论线程以获取上下文
• 建立帖子清单，以便后续手动查看/回复

> 这是只读的。不支持发帖、投票或评论。

> **👤 人机分工（最小必要人工 · 时点 · 凭证）**
> - **一次性（开始前）**：提供想订阅的子版块列表（`<paste the list here>`）。
> - **付费 / 门槛**：无（技能免费、无需认证）。
> - **周期性 / 自动**：cron 每天下午 5 点自动拉取并生成摘要；每天由 agent 询问你是否满意，自动把喜好写入偏好记忆，人工仅需回答喜好反馈。
> - **外发前确认**：无需确认 —— 纯只读，不发帖、不投票、不评论，无任何对外动作。
> - **凭证**：无（技能无需认证）
>
> 装 `reddit-readonly` 技能 / 建 cron 等能力项不列在此——由你的 agent 现场探测并代办（协议见 [AGENTS.md](../AGENTS.md)）。

## 所需技能
[reddit-readonly](https://clawhub.ai/buksan1950/reddit-readonly) 技能。无需认证。

## 如何设置
安装该技能后，向你的 OpenClaw 发送以下提示：

以下提示词可直接使用：
```text
I want you to give me the top performing posts from the following subreddits.
<paste the list here>
Create a separate memory for the reddit processes, about the type of posts I like to see and every day ask me if I liked the list you provided. Save my preference as rules in the memory to use for a better digest curation. (e.g. do not include memes.)
Every day at 5pm, run this process and give me the digest.
```

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/daily-reddit-digest.md)
