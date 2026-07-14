# 自主教育游戏开发流水线

> 含国内适配：微信小游戏 / Cocos Creator / TapTap

一位老派 LAN 玩家父亲，想为女儿们打造一个安全、无广告、高质量的游戏门户。现有网站充斥着垃圾信息、弹窗广告和欺骗性按钮（暗黑模式），连三岁的孩子都会被误导点击。

## 痛点

搭建一个"干净、快速、简洁"的游戏门户并不难，真正的挑战在于——如何在没有开发团队的情况下，为 0-15 岁不同发育阶段的孩子填充 **40+ 款教育游戏**？手动开发对于一个独立开发者来说太慢了，而且维护几十款游戏的一致性更是噩梦。

## 功能概述

这个用例定义了一个"游戏开发智能体"，自主管理游戏创建和维护的整个生命周期。工作流执行严格的 **"Bug 优先"** 策略——智能体必须先检查并解决已报告的 Bug，然后才能开发新功能。

**效率**：这条流水线能达到 **每 7 分钟产出 1 个新游戏或修复一个 Bug** 的速度。智能体不知疲倦地遍历 41+ 个计划中的游戏 Backlog，在创建新内容和修正之前发现的问题之间交替进行。

当路径清晰时，智能体会：
1. **选择**：根据"轮询"策略从队列（`development-queue.md`）中选取下一个游戏，以平衡各年龄段的内容
2. **实现**：按照严格的 `game-design-rules.md` 编写 HTML5/CSS3/JS 游戏代码（不使用框架、移动端优先、支持离线）
3. **注册**：自动将游戏元数据添加到中央注册表（`games-list.json`）
4. **文档化**：更新 `CHANGELOG.md` 和 `master-game-plan.md` 状态
5. **部署**：处理完整的 Git 工作流：拉取 master、创建 feature 分支、提交规范化 commit、合并回主分支

## 提示词

以下是给智能体的核心**系统指令**。这段提示词将 LLM 变成一个遵守项目严格结构的纪律化开发者。

*（**注意**：原始生产环境中的提示词使用**西班牙语**（es-419），以契合项目的目标受众——拉丁美洲儿童。以下版本已翻译为英文供参考使用。）*

```text
Act as an Expert in Web Game Development and Child UX.
Your goal is to develop the next game in the production queue.

Please read and analyze the following context files before starting:

1.  BUG CONTEXT (Top Priority - CRITICAL):
    @[bugs/]
    (Check this folder. If there are files, YOUR TASK IS TO FIX **ONLY THE FIRST FILE** (in alphabetical order). Ignore the rest of the bugs and the game queue for now).

2.  QUEUE CONTEXT (Which game is next):
    @[development-queue.md]
    (Identify the game marked as [NEXT] in the "Next Games" section. ONLY if there are no bugs).

3.  DESIGN RULES (Technical Standards):
    @[game-design-rules.md]
    (Strictly follow these rules: Pure HTML/CSS/JS, folder structure, mobile responsiveness)

4.  GAME SPECIFICATIONS (Mechanics and Assets):
    (Identify the corresponding file in games-backlog/ based on the game ID)

5.  CENTRAL REGISTRY (Integration):
    @[public/js/games-list.json]
    (File where you MUST register the new game so it appears on the home page)

TASK:
0. **BUGS FIRST!**: If the `bugs/` folder has content, your only priority is to fix **the first bug in alphabetical order**. Create a `fix/...` branch, resolve **that** bug, update status, and merge. **Do not attempt to fix multiple bugs at once.**
   - IF THERE ARE NO BUGS, proceed with the next game:

1. **Synchronization**: `git fetch && git pull origin master` (CRITICAL).
2. Create a new branch: `git checkout -b feature/[game-id]`.
3. Create the folder and files in 'public/games/[game-id]/'.
4. Implement logic and design according to the backlog and design rules.
5. Register the game in 'games-list.json' (CRITICAL).
6. When finished:
    - Update `CHANGELOG.md` bumping the version.
    - Update `master-game-plan.md` and `development-queue.md`.
    - Document changes: `git commit -m "feat: add [game-id]"`.
7. **Delivery**:
    - Push: `git push origin feature/[game-id]`.
    - Request merge to master.
    - Once in master, push changes (`git push origin master`).
```

## 所需技能

- **Git**：用于管理分支、提交和合并

## 关键洞察

- **"Bug 优先"策略**是关键——强制智能体先修复问题再开发新功能，确保质量不会随着游戏数量增长而下降
- **轮询策略**确保各年龄段的内容均衡发展，不会只偏向某一个年龄组
- 纯 HTML5/CSS3/JS 的技术约束（无框架）使得游戏轻量、快速加载，且便于智能体生成和维护
- 项目的文件结构严格规范化（队列、Backlog、设计规则、注册表），这是自主开发流水线能稳定运行的基础
- 智能体可以循环运行——每次执行结束后，再次启动即可继续处理队列中的下一个任务

## 中国用户适配

### 场景契合度

这个用例对中国用户有很强的实用价值：

- **儿童教育游戏需求旺盛**：国内家长对安全、无广告的儿童教育内容有强烈需求，商业平台的广告和付费墙让很多家长头疼
- **独立游戏社区活跃**：TapTap 月活超 4300 万，国内 Steam 游戏开发者数量逐年增长
- **H5 游戏生态成熟**：微信小游戏、抖音小游戏等平台让 HTML5 游戏有了天然的分发渠道

### 国内替代方案

| 原版组件 | 国内替代 | 说明 |
|---------|---------|------|
| 纯 HTML5/CSS3/JS | Cocos Creator / Laya | 如需更强的 2D/3D 能力，可使用国产引擎；纯 H5 方案同样适用 |
| GitHub Pages 部署 | Gitee Pages / 腾讯云 Webify | 国内访问速度更快 |
| 独立网站分发 | 微信小游戏 / 抖音小游戏 | 自带流量入口，触达更多用户 |
| Git 工作流 | 相同 | Git 是通用工具，无需替换 |

### 适配建议

1. **内容本地化**：将游戏内容改为中文，教育内容对齐国内课程标准（如识字顺序、拼音教学等）
2. **部署到小游戏平台**：纯 HTML5 游戏可以较容易地适配为微信小游戏，获得微信生态的分发优势
3. **合规注意**：国内发布儿童向内容需关注《未成年人保护法》相关规定，确保内容适龄
4. **国内镜像**：如果使用 GitHub，建议同步到 Gitee 作为国内镜像，或直接使用腾讯云/阿里云部署

## 相关链接

- [项目诞生故事 (LinkedIn)](https://www.linkedin.com/feed/update/urn:li:activity:7429739200301772800/) — 配置 OpenClaw 后如何催生了这个项目
- [El Bebe Games 源码仓库](https://github.com/duberblockito/elbebe/tree/master) — 源代码
- [El Bebe Games 在线网站](https://elbebe.co/) — 流水线的成果展示
- [HTML5 游戏开发最佳实践 (MDN)](https://developer.mozilla.org/zh-CN/docs/Games)

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/autonomous-game-dev-pipeline.md)
