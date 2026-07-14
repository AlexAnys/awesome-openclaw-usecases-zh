# Adam Framework：OpenClaw 智能体持久记忆

> 含国内适配：Kimi K2.5 / 通义千问 / 智谱 GLM / 离线部署方案

每次开启新的 OpenClaw 会话，你的智能体都是一片空白。你得反复解释你的项目、业务、目标和决策。用了好几周，它连你的名字都不知道——除非你主动告诉它。这就是 AI 失忆症，而且你越深入使用，累积的损耗越大。

还有一个更隐蔽的问题：在长对话中，随着上下文累积，智能体的推理一致性会悄无声息地下降——它不会主动告诉你，只是开始"漂移"。

Adam Framework 通过 5 层架构，在本地为你的 OpenClaw 智能体赋予跨会话的持久记忆和身份一致性。

## 痛点

大多数人尝试的解决办法：

- **每次开头复制粘贴上下文** — 费时、不完整、无法扩展
- **依赖 AI 内置记忆功能** — 浅层、不可靠、经常出错
- **每次重新开始** — 违背了使用 AI 的初衷

Adam Framework 在架构层面同时解决两个问题：**跨会话失忆** 和 **会话内漂移**。

## 它能做什么

### 5 层架构

| 层级 | 功能 | 说明 |
|------|------|------|
| **Layer 1：Vault 注入** | 身份文件注入 | SENTINEL 在每次启动时加载 SOUL.md 和 CORE_MEMORY.md，智能体醒来就知道自己是谁 |
| **Layer 2：会话检索** | 中途记忆搜索 | 混合搜索，通过 memory_search / memory_get 工具在对话中实时检索 |
| **Layer 3：神经图谱** | 关联记忆 | nmem_context 提供联想式回忆，7,219 个神经元、29,291 个突触的知识图谱 |
| **Layer 4：夜间整合** | 睡眠周期 | 使用 Gemini（或国内 API 替代）在你睡觉时将日志合并到 CORE_MEMORY.md |
| **Layer 5：一致性监控** | 漂移检测 | 每 5 分钟检查 scratchpad 使用情况，检测到漂移时自动重新锚定 |

### 体验变化

- **第 1 天**：智能体在你开口之前就知道你的名字、项目和角色。会话从有上下文的状态开始。
- **第 1 周**：神经图谱建立了真实的关联。智能体开始主动引用之前会话的内容。
- **第 1 个月**：睡眠周期已合并数周的日志。智能体拥有了真正的项目历史。记忆开始复利增长。

## 所需技能

- OpenClaw 已安装并运行，连接了一个模型
- Python 3.10+
- mcporter（`npm install -g mcporter`）
- NVIDIA Developer 免费 API 密钥（Kimi K2.5，131K 上下文，免费额度）
- Gemini API 密钥（免费）— 用于夜间睡眠周期

## 如何设置

有两种安装路径：

### 路径 1：手动安装（30-60 分钟）

按照 [SETUP_HUMAN.md](https://github.com/strangeadvancedmarketing/Adam/blob/master/SETUP_HUMAN.md) 的四个阶段操作：

**阶段 1 — 赋予身份（30 分钟）**

1. 创建 Vault 目录（智能体的记忆存储位置）：

```bash
# macOS/Linux
mkdir -p ~/MyAIVault/workspace/memory
mkdir -p ~/MyAIVault/imports
```

2. 从模板创建身份文件：

```bash
cp vault-templates/SOUL.template.md ~/MyAIVault/SOUL.md
cp vault-templates/CORE_MEMORY.template.md ~/MyAIVault/CORE_MEMORY.md
cp vault-templates/BOOT_SEQUENCE.md ~/MyAIVault/workspace/BOOT_SEQUENCE.md
cp vault-templates/active-context.template.md ~/MyAIVault/workspace/active-context.md
```

3. 编辑 SOUL.md，填写所有 `{{PLACEHOLDER}}`：
   - `{{YOUR_AI_NAME}}` — 你想给 AI 起的名字
   - `{{YOUR_NAME}}` — 你的名字
   - `{{YOUR_ROLE}}` — 你的工作角色
   - `{{YOUR_AI_PERSONALITY}}` — 你希望它如何沟通
   - `{{YOUR_BUSINESS_CONTEXT}}` — 你的项目和业务

4. 配置 openclaw.json，添加 Vault 路径：

```json
"extraPaths": [
  "/Users/你的用户名/MyAIVault/CORE_MEMORY.md",
  "/Users/你的用户名/MyAIVault/workspace/memory",
  "/Users/你的用户名/MyAIVault/workspace/BOOT_CONTEXT.md"
]
```

5. 设置 SENTINEL 看门狗脚本：

```bash
cp engine/SENTINEL.template.sh ~/.openclaw/SENTINEL.sh
chmod +x ~/.openclaw/SENTINEL.sh
# 编辑 SENTINEL.sh，设置 VAULT_PATH="$HOME/MyAIVault"
```

**阶段 2 — 安装记忆系统（15 分钟）**

```bash
pip install neural_memory
# 配置 mcporter 路由
mcporter call "neural-memory.nmem_stats()"
# 预期输出：{"neurons": 0, "synapses": 0}
```

**阶段 3 — 导入历史对话（15 分钟 + 后台运行）**

导出你在 Claude / ChatGPT 的对话历史，提取事实并注入神经图谱：

```bash
python tools/legacy_importer.py \
    --source ~/Downloads/claude-export.zip \
    --vault-path ~/MyAIVault \
    --user-name "你的名字"
```

**阶段 4 — 自动化运维**

将工具复制到 Vault 目录，配置 Gemini API 密钥，设置 SENTINEL 开机自启：

```bash
# macOS
cp engine/com.adamframework.sentinel.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.adamframework.sentinel.plist
```

### 路径 2：AI 辅助安装

将 [SETUP_AI.md](https://github.com/strangeadvancedmarketing/Adam/blob/master/SETUP_AI.md) 的内容粘贴到你的智能体对话中。它会问你 8 个问题，然后自动完成安装。

## 验证数据

在一个真实商业场景中经过 353 次会话、6,619 轮对话验证。一致性监控通过 33/33 项测试。交互式验证页面：[AI Amnesia Solved](https://strangeadvancedmarketing.github.io/Adam/showcase/ai-amnesia-solved.html)

## 关键洞察

- **身份文件是最重要的投入**：花 30 分钟认真填写 SOUL.md 和 CORE_MEMORY.md，这个投入会在每次会话中复利
- **从简单开始**：可以先只用 Layer 1（Vault 注入）和 Layer 2（会话检索），验证效果后再逐步启用 Layer 3-5
- **记忆是 Markdown 文件**：人类可读、可编辑、可 Git 追踪、兼容 Obsidian，每个文件都可以审计
- **睡眠周期不是必须的**：如果不想依赖外部 API，可以手动维护 CORE_MEMORY.md
- **一致性监控解决的是隐性问题**：长对话中的推理漂移很难被察觉，Layer 5 每 5 分钟自动检测并纠正

> 如果只需要对现有 Markdown 文件添加语义搜索能力，可参考更轻量的 [语义记忆搜索](semantic-memory-search.md) 方案。Adam Framework 提供完整的 5 层记忆架构，适合需要跨会话身份一致性和自动记忆整合的深度用户。

## 中国用户适配

### 核心挑战

Adam Framework 的 Layer 4（夜间整合）依赖 Gemini API，而 Gemini 在国内需要网络代理。以下方案帮你用国内 API 完全替代，或者直接跳过 Layer 4 实现离线部署。

### API 替代方案

| 原版依赖 | 国内替代 | 费用 | 说明 |
|---------|---------|------|------|
| **Gemini API**（Layer 4 睡眠周期） | **Kimi K2.5** | 输入 4 元/百万 token，输出 21 元/百万 token | 通过 Moonshot API 调用，性能对标 Gemini，开源可审计 |
| | **通义千问 qwen-max** | 按量计费，新用户有免费额度 | 阿里云百炼平台，企业用户首选 |
| | **智谱 GLM-4-Plus** | 按量计费，免费额度充足 | 智谱 AI 开放平台，中文理解能力强 |
| **NVIDIA Free API**（主模型） | **Kimi K2.5** | OpenClaw 官方补贴免费 | OpenClaw 已将 Kimi K2.5 列为首个开源主力模型 |
| | **通义千问 qwen-max** | 按量计费 | 通过阿里云百炼平台接入 |

**推荐方案**：主模型和睡眠周期都用 **Kimi K2.5**，既免费又是目前中文场景下性能最好的开源模型之一。

### 替换 Layer 4 睡眠周期的 Gemini 调用

`reconcile_memory.py` 脚本中调用 Gemini 的部分需要替换为国内 API。核心改动：

```python
# 原版：使用 Gemini API
# import google.generativeai as genai
# genai.configure(api_key=os.environ["GEMINI_API_KEY"])
# model = genai.GenerativeModel("gemini-2.0-flash")

# 替换为 Kimi K2.5（OpenAI 兼容格式）
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["MOONSHOT_API_KEY"],
    base_url="https://api.moonshot.cn/v1"
)

def reconcile_logs(daily_log: str, core_memory: str) -> str:
    response = client.chat.completions.create(
        model="kimi-k2.5",  # 请通过 GET /v1/models 确认当前可用模型 ID
        messages=[
            {"role": "system", "content": "你是一个记忆整合助手。将以下日志内容合并到核心记忆中，保留关键决策、项目状态和重要事实。删除冗余信息。输出更新后的核心记忆内容。"},
            {"role": "user", "content": f"当前核心记忆：\n{core_memory}\n\n今日日志：\n{daily_log}"}
        ]
    )
    return response.choices[0].message.content
```

在 openclaw.json 的 `env` 块中配置密钥：

```json
"env": {
  "MOONSHOT_API_KEY": "你的-Moonshot-API-密钥"
}
```

> 获取密钥：[Moonshot 开放平台](https://platform.moonshot.cn/)，注册即可获得免费额度。

**通义千问替代方案**：

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DASHSCOPE_API_KEY"],
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)
# model 改为 "qwen-max" 即可，其余代码不变
```

### 离线部署方案（不依赖任何外部 API）

如果你不想依赖任何云端 API（出于安全或网络原因），可以只部署 Layer 1 + 2 + 3 + 5，跳过 Layer 4 的自动睡眠周期：

| 层级 | 是否需要 | 离线替代方案 |
|------|---------|-------------|
| Layer 1：Vault 注入 | 需要 | 纯本地 Markdown 文件，无外部依赖 |
| Layer 2：会话检索 | 需要 | OpenClaw 内置功能，纯本地 |
| Layer 3：神经图谱 | 需要 | neural_memory 本地 SQLite，无外部依赖 |
| Layer 4：夜间整合 | **可跳过** | 手动更新 CORE_MEMORY.md，或用本地模型（Ollama + Qwen2.5） |
| Layer 5：一致性监控 | 需要 | Python 脚本，纯本地运行 |

使用 Ollama 运行本地模型替代 Layer 4：

```bash
# 安装 Ollama
curl -fsSL https://ollama.com/install.sh | sh
# 拉取通义千问开源模型
ollama pull qwen2.5:14b

# 修改 reconcile_memory.py，使用 Ollama 的 OpenAI 兼容接口
```

```python
from openai import OpenAI

client = OpenAI(
    api_key="ollama",  # Ollama 不需要真实密钥
    base_url="http://localhost:11434/v1"
)
# model 改为 "qwen2.5:14b"，其余代码不变
```

### 中文 SOUL.md 模板示例

```markdown
# 灵魂定义

## 身份
- 名称：小智
- 角色：{{YOUR_NAME}} 的个人 AI 助手
- 性格：务实、高效、有条理，说话简洁有重点

## 运营者
- 姓名：{{YOUR_NAME}}
- 角色：{{YOUR_ROLE}}
- 沟通偏好：中文为主，技术术语保留英文

## 核心原则
1. 每次回复前使用 scratchpad 思考（保证推理一致性）
2. 主动引用过往对话中的决策和上下文
3. 不确定时坦诚说明，不编造记忆
4. 优先使用运营者偏好的工具和平台

## 边界
- 不做未经确认的外部操作（发邮件、发消息等）
- 涉及敏感信息时主动提醒
- 单次对话超过 2 小时时主动建议休息
```

### 中文 CORE_MEMORY.md 模板示例

```markdown
# 核心记忆

> 此文件由睡眠周期自动更新。你也可以手动编辑。

## 当前项目

### 项目 A：{{项目名称}}
- 状态：进行中
- 关键决策：
  - {{日期}}：决定使用 XX 技术方案
- 当前阻塞：无
- 下一步：{{具体任务}}

## 关键联系人
- {{名字}}：{{角色/关系}}，偏好通过 {{平台}} 沟通

## 工具和偏好
- 主力 IDE：VS Code
- 项目管理：飞书项目
- 消息平台：飞书 / 钉钉 / 企业微信（按实际情况填写）

## 重要事实
- {{日期}}：{{发生了什么，为什么重要}}
```

### 与国内 IM 平台集成

Adam Framework 的 Vault 记忆系统可以与国内 IM 平台结合使用，让你从飞书/钉钉/企业微信直接与拥有持久记忆的智能体对话：

| 平台 | 集成方式 | 参考用例 |
|------|---------|---------|
| **飞书** | 通过飞书渠道插件连接 OpenClaw，Vault 注入 + 神经图谱在后台运行 | [飞书 AI 助手](cn-feishu-ai-assistant.md) |
| **钉钉** | 通过钉钉 Stream 模式连接，无需公网 IP | [钉钉 AI 助手](cn-dingtalk-ai-assistant.md) |
| **企业微信** | 通过企业微信应用连接 | [企业微信 AI 助手](cn-wecom-ai-assistant.md) |

核心思路：IM 平台只是通信渠道，Adam Framework 的记忆系统在 OpenClaw 后端运行。无论从哪个渠道发消息，智能体都能访问同一套持久记忆。

### 故障排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 智能体不认识我 | BOOT_CONTEXT.md 未编译 | 检查 SENTINEL 脚本中的 VAULT_PATH 是否正确 |
| nmem_stats() 无响应 | mcporter 找不到 neural-memory 服务 | 运行 `python -m neural_memory.mcp_server` 确认无报错 |
| 睡眠周期未运行 | API 密钥未配置 | 检查 openclaw.json 的 env 块中是否有 MOONSHOT_API_KEY |
| Gateway 崩溃循环 | openclaw.json 格式错误 | 用 [jsonlint.com](https://jsonlint.com) 检查 JSON 格式 |
| 导入的事实太少 | 对话记录太短 | 尝试 `--max-per-conv 50` 参数进行更积极的提取 |

### 安全提醒

- SOUL.md 和 CORE_MEMORY.md 包含个人和业务信息，建议**不要**推送到公开仓库
- 如果使用 Git 追踪 Vault，确保仓库为**私有**
- API 密钥通过环境变量或 openclaw.json 的 `env` 块传递，不要硬编码在脚本中
- 离线部署方案适合对数据安全要求高的场景（金融、医疗等行业）

## 相关链接

- [Adam Framework 仓库](https://github.com/strangeadvancedmarketing/Adam)
- [SETUP_HUMAN.md — 手动安装指南](https://github.com/strangeadvancedmarketing/Adam/blob/master/SETUP_HUMAN.md)
- [SETUP_AI.md — AI 辅助安装](https://github.com/strangeadvancedmarketing/Adam/blob/master/SETUP_AI.md)
- [架构文档](https://github.com/strangeadvancedmarketing/Adam/blob/master/docs/ARCHITECTURE.md)
- [生产环境经验教训](https://github.com/strangeadvancedmarketing/Adam/blob/master/docs/LESSONS_LEARNED.md)
- [Moonshot 开放平台](https://platform.moonshot.cn/) — Kimi K2.5 API 接入
- [阿里云百炼](https://dashscope.aliyun.com/) — 通义千问 API 接入
- [智谱 AI 开放平台](https://open.bigmodel.cn/) — GLM-4 API 接入
- [Ollama](https://ollama.com/) — 本地运行开源模型

---

**原文链接**：[English Version](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/56)
