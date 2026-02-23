# 收件箱整理

新闻订阅邮件（newsletter）会迅速占满收件箱。它们经常堆积如山，根本没有被打开过。

## 所需技能
[Gmail OAuth 设置](https://clawhub.ai/kai-jar/gmail-oauth)。

## 如何设置
1. [可选] 为 OpenClaw 专门创建一个新的 Gmail 账户。
2. [可选] 取消主邮箱中所有新闻订阅，改用 OpenClaw 邮箱订阅。
3. 安装技能并确保其正常工作。
4. 指示 OpenClaw：

以下提示词让 OpenClaw 每天自动整理新闻摘要：

```txt
I want you to run a cron job everyday at 8 p.m. to read all the newsletter emails of the past 24 hours and give me a digest of the most important bits along with links to read more. Then ask for my feedback on whether you picked good bits, and update your memory based on my preferences for better digests in the future jobs.
```

该提示词要求 OpenClaw 设置一个定时任务（cron job），每天晚上 8 点读取过去 24 小时内的所有新闻订阅邮件，提供最重要内容的摘要和阅读链接，然后根据你的反馈更新记忆，以便未来生成更好的摘要。

---

**原文链接**：[English Version](https://github.com/AlexAnys/awesome-openclaw-usecases/blob/main/usecases/inbox-declutter.md)
