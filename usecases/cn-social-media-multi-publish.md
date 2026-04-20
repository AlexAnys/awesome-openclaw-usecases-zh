# 社交媒体全平台一键发布

自媒体运营者的日常：同一条视频要分别打开抖音创作者中心、B 站投稿页、小红书后台、快手发布页，逐个上传、填标题、选封面、设定时——四个平台走一遍至少半小时，多账号翻倍。内容生产已经够累了，分发不该再吃掉这么多时间。

这个用例通过社区成熟的自动化引擎 [social-auto-upload](https://github.com/dreammis/social-auto-upload)（9,800+ stars），将抖音、B 站、小红书、快手、视频号、百家号的登录和发布打通成统一的命令行（CLI）操作，再配合 OpenClaw 的自然语言调度，实现"说一句话，全平台发出去"。

> **与已有用例的区别**：
> - [小红书内容自动化](cn-xiaohongshu-automation.md)——专注小红书单平台的选题、文案、封面、发布全流程，使用 XiaohongshuSkills
> - 本用例——专注**跨平台统一分发**，用一个引擎覆盖 6 个已可用平台，解决的是"同一份内容怎么高效铺开"的问题

## 它能做什么

- **统一登录管理**：每个平台扫码/浏览器登录一次，cookie 自动持久化，下次无需重复
- **多平台视频发布**：一条命令将视频上传到抖音、B 站、小红书、快手等平台
- **图文发布**：小红书、快手、抖音支持图文内容上传
- **定时发布**：指定发布时间，错峰分发到各平台
- **AI 文案适配**：配合内容生成工具，针对各平台字数限制和风格差异生成适配文案
- **Cookie 状态检测**：登录凭证过期时自动提醒重新扫码

## 核心引擎

[social-auto-upload](https://github.com/dreammis/social-auto-upload) —— 基于 Playwright（浏览器自动化框架）的多平台上传工具，社区活跃维护，2026 年 3 月仍有密集提交。

| 平台 | 视频 | 图文 | 定时 | 登录方式 | 状态 |
|------|:---:|:---:|:---:|---------|------|
| 抖音 | ✅ | ✅ | ✅ | 手机扫码 | 可用 |
| B 站 | ✅ | - | ✅ | 手机扫码 | 可用 |
| 小红书 | ✅ | ✅ | ✅ | 浏览器手动登录 | 可用 |
| 快手 | ✅ | ✅ | ✅ | 浏览器手动登录 | 可用 |
| 视频号 | ✅ | - | ✅ | 微信扫码 | 可用 |
| 百家号 | ✅ | - | ✅ | 浏览器手动登录 | 可用 |

需要 Python 3.10+ 和 Chromium 浏览器（Playwright 自动安装）。

## 如何设置

### 第一步：安装引擎

```bash
# 克隆仓库
git clone https://github.com/dreammis/social-auto-upload.git
cd social-auto-upload

# 推荐用 uv 管理环境（也可以用 pip）
uv sync

# 安装浏览器驱动（uv 环境下用 uv run 调用）
uv run playwright install chromium
```

> 也可以用传统 venv：`python3 -m venv .venv && source .venv/bin/activate && pip install -e . && playwright install chromium`，之后命令不需要 `uv run` 前缀。

验证安装：

```bash
# uv 环境
uv run sau --help
# 或已激活的 venv
sau --help
```

### 第二步：登录各平台

每个平台只需登录一次，cookie 自动保存到 `cookies/` 目录（以下命令在 uv 环境下需加 `uv run` 前缀，激活 venv 后可直接执行）：

```bash
# 抖音——终端弹出二维码，用手机抖音扫码
sau douyin login

# B 站——手机 B 站 APP 扫码
sau bilibili login

# 小红书——自动打开浏览器，手动在创作者中心登录
sau xiaohongshu login

# 快手——同小红书，浏览器手动登录
sau kuaishou login
```

> 各平台 cookie 有效期不同：抖音约 30 天，B 站约 180 天，小红书/快手视平台策略而定。过期后重新执行 `login` 命令扫码即可。

### 第三步：发布内容

```bash
# 单平台视频发布
sau douyin upload --video video.mp4 --title "周末日常" --tags "生活,vlog"
sau bilibili upload --video video.mp4 --title "周末日常"

# 小红书图文发布
sau xiaohongshu upload --images img1.jpg img2.jpg --title "居家好物分享" --desc "最近发现的几个实用小物件"

# 定时发布（指定发布时间）
sau douyin upload --video video.mp4 --title "周末日常" --schedule "2026-04-12 20:00"
```

### 第四步：配合 OpenClaw 使用

将 social-auto-upload 所在目录加入 OpenClaw 的 skills 路径（如 `~/.openclaw/workspace/skills/`），让 OpenClaw 能识别并调用 `sau` 命令。配置完成后，用自然语言发布：

```text
帮我把 video.mp4 发到抖音和 B 站，标题"周末日常"，抖音加上标签"生活,vlog"

全平台发布 output.mp4，标题各平台适配一下，抖音定时今晚 8 点

帮我登录小红书，我之前的 cookie 过期了
```

OpenClaw 会自动构造对应的 `sau` 命令并执行。

## 实用建议

- **逐平台验证再批量**：先在一个平台跑通全流程，确认视频格式、标题长度、标签规则都没问题，再扩展到其他平台
- **标题和描述差异化**：完全相同的标题发到多个平台容易被判定搬运，建议每个平台至少调整标题措辞和标签
- **控制发布节奏**：避免短时间内向同一平台密集上传，每次发布间隔 15-30 分钟更安全；每账号每天建议不超过 5 条
- **定时发布需保持在线**：定时功能依赖本机运行，电脑休眠或断网会导致发布失败
- **二维码登录排错**：如果扫码失效，先 `git pull` 更新引擎（平台改接口是最常见原因），然后清除 `cookies/` 目录重试。终端二维码显示异常时把窗口拉大或用参数保存为图片
- **B 站额外依赖**：B 站上传底层依赖 [biliup](https://github.com/biliup/biliup)，如果 B 站上传异常，尝试 `pip install --upgrade biliup`

> **⚠️ 平台风控提醒**：抖音、B 站、小红书、快手、视频号等平台均对自动化操作有风控策略，包括但不限于浏览器指纹检测、发布频率限制、内容重复度识别、设备/IP 异常检测。自动化操作可能导致账号限流、降权甚至封禁。**强烈建议使用测试账号充分验证后再用于正式账号**，以提升分发效率为目标，不要用于批量灌水或违规内容发布。各平台 cookie 过期后需手动重新扫码，无法绕过。

## 相关链接

- [social-auto-upload - GitHub](https://github.com/dreammis/social-auto-upload) — 核心引擎（9,800+ stars）
- [知乎 - 免费开源神器：一键分发，自动化短视频上传](https://zhuanlan.zhihu.com/p/699662685)
- [知乎 - 开源推荐：Social Auto Upload](https://zhuanlan.zhihu.com/p/1892997377016698512)
- [CSDN - Social-Auto-Upload：跨平台视频自动发布神器](https://blog.csdn.net/j8267643/article/details/151585936)
- [cnblogs - 多平台社交媒体视频自动化上传工具](https://www.cnblogs.com/wzzkaifa/p/19122437)
- [NasDaddy - 全网发遍视频图文](https://nasdaddy.com/2026-sau-upload/)
- [social-media-publish-skill](https://github.com/huangji6693-max/social-media-publish-skill) — 社区封装的 OpenClaw Skill 包（新建，可关注后续发展）
