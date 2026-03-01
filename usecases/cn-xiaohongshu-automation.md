# 小红书内容自动化

自媒体运营者每天在小红书上的工作流程：追热点 → 写文案 → 做封面图 → 排期发布。每篇笔记从构思到发布至少一小时，如果要日更多个账号，时间直接翻倍。

这个用例让 OpenClaw 完成从选题、文案撰写、封面图生成到定时发布的全流程自动化。

## 它能做什么

- **热点检测**：自动分析小红书当前热门话题
- **文案生成**：AI 按照小红书特有的写作风格生成图文笔记（包括标题、正文、标签）
- **封面图生成**：调用 AI 图像模型生成专业封面图
- **自动发布**：登录账号后定时发布笔记
- **数据追踪**：查看笔记的浏览、点赞、收藏数据

## 所需技能

[XiaohongshuSkills](https://github.com/white0dew/XiaohongshuSkills) —— 社区开发的小红书自动化技能，支持 OpenClaw 直接调用。

需要 Python 3.10+ 和 Chrome 浏览器。

## 如何设置

1. 克隆项目并安装依赖：
```bash
git clone https://github.com/white0dew/XiaohongshuSkills.git
cd XiaohongshuSkills
pip install -r requirements.txt
```

2. 登录账号（首次需要扫码）：
```bash
python scripts/cdp_publish.py login
```

3. 用自然语言发布内容：
```text
生成一篇美妆推荐的图文笔记，明天上午 10 点发布。

帮我写一篇关于"居家收纳技巧"的小红书笔记，要有吸引力的标题和 5 个相关话题标签。

分析一下最近"健身餐"相关的热门笔记，找出高赞共性。
```

## 实用建议

- **内容质量优先**：AI 生成的内容建议人工审核后再发布，确保内容质量和品牌调性一致
- **控制发布频率**：过于频繁的自动发布可能触发平台风控，建议每天不超过 3-5 篇
- **多账号管理**：该技能支持多账号隔离的 Cookie 管理，可以同时管理多个运营账号
- **数据驱动迭代**：用数据追踪功能分析哪类内容表现好，让 OpenClaw 记住你的偏好并持续优化选题

> **⚠️ 注意**：小红书对自动化操作有风控策略。请遵守平台使用规范，建议以辅助创作为主，避免纯粹的批量灌水。

## 相关链接

- [XiaohongshuSkills - GitHub](https://github.com/white0dew/XiaohongshuSkills)
- [CSDN - 实战案例：用 OpenClaw 自动发小红书笔记](https://blog.csdn.net/weixin_49598732/article/details/158286134)
- [腾讯云 - 把小红书交给 OpenClaw](https://cloud.tencent.com/developer/article/2629870)
