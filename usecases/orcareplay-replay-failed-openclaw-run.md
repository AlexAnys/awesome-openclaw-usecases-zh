# OrcaReplay 复现一次失败的 OpenClaw 运行

OpenClaw 把 coding agent 当子进程拉起：一次任务里既有网关自己发出的一轮轮模型请求，也有它拉起来的 Claude Code / Codex / opencode 发出的请求。凌晨两点跑坏了一次任务，早上九点回来看，终端已经滚过去了 —— 再跑一遍大概率得到另一个失败，而 provider 边界上真实发生过的那些请求，在会话结束的那一刻就已经消失了。

这个用例用 OrcaReplay 把整次 OpenClaw 运行录成本地 trace，之后断网、逐字节地把它重放出来，定位到真正出错的那一步；再从出事之前的最后一个 checkpoint 换一个模型重试，看换模型能不能做对。

## 痛点

- **失败不可复现**：重跑得到的是另一个结果，原来的现场没有留存任何证据
- **两层调用混在一起**：网关自身的调用和它拉起的 coding agent 的调用在同一时间发生，日志里很难分清谁发的
- **看不到模型之外发生了什么**：真正落到文件系统上的改动、shell 命令的真实退出码，模型 transcript 里都没有 —— agent 的退出码是 0，不代表它跑的那个检查是成功的
- **回答不了「换个模型会不会做对」**：没有相同的起点，所谓重跑就是新的一次运行

## 它能做什么

- `orca record openclaw` 录制整次运行：网关自身的调用由 fetch hook 捕获；被拉起的 coding agent 的调用由环境变量捕获 —— 不是因为 OpenClaw 读这些变量，而是因为子进程继承父进程的环境，它 spawn 的 Claude Code 看到 `ANTHROPIC_BASE_URL` 的方式，和你自己跑它完全一样
- `orca replay last` 在不联网、不调用模型、不花钱的前提下按字节重放这次运行
- `orca show last` 给出这次运行按时间发生了什么，`orca graph last` 给出谁导致了谁
- `orca compare last --from N --models a,b --verify "<cmd>"` 从任意 checkpoint 换模型重跑，并用你给的命令逐个打分
- `orca export last -o bug.html` 导出一个自包含的 HTML 文件，可以直接贴到 issue 里
- `orca mcp` 把本地 trace 库以 MCP server 的形式挂出去（list / show / checkpoints / graph / replay / compare 六个工具），让 agent 自己读自己的历史运行

## 所需技能

- 已安装并能正常使用的 OpenClaw，以及它拉起的 coding agent
- Node 20+
- 无需额外 OpenClaw skills；OrcaReplay 是独立的 Node CLI（`npm i -g orcareplay`），Apache-2.0

## 如何设置

### 1. 安装

```bash
npm i -g orcareplay
orca doctor      # 检查 node / git，以及本机上能找到哪些 agent
```

### 2. 录制

像平时一样启动 OpenClaw，只是把它放到录音机后面：

```bash
orca record openclaw
```

### 3. 复现你的任务

让 OpenClaw 完整跑一遍那个会出错的工作流，然后退出。

### 4. 先看这张顺序表

下面这三段输出取自项目文档，用来说明各命令的输出形态；你跑出来的会是自己的 run id 和自己的事件序列。

```console
$ orca show last
run_6473f858b59e  openclaw@0.1.0  14 events  exit 0

SEQ  KIND   WHAT                                            DETAIL
0    RUN    run started                                     generic-openai
1    SNAP   tree 919d32ba037537b43814c83779963b2cc3023db7   0 changed
2    MODEL  claude-opus-5                                   1 messages
3    MODEL  claude-opus-5                                   stop: tool_use · 100 in · 20 out
4    TOOL   edit_file                                       {"path":"auth.ts",…}
5    SNAP   tree c6af62b75c0c8b8938bd6087328b5148f3dcd534   1 changed
6    FILE   auth.ts                                         modified +1 −3
7    TOOL   edit_file                                       ok
8    MODEL  claude-opus-5                                   3 messages
9    MODEL  claude-opus-5                                   stop: end_turn · 101 in · 5 out
10   SNAP   tree c6af62b75c0c8b8938bd6087328b5148f3dcd534   0 changed
11   SHELL  ["sh","-c","node --check nonexistent-file.ts"]  /tmp/hunt
12   SHELL  shell result                                    exit 1 · 43ms
13   RUN    run ended                                       exit 0

info usage input=201 output=25 cost=$0.004890
```

上面有三件事是模型的 transcript 讲不出来、退出码也藏起来的：文件确实被改了（seq 6，`+1 −3`）；agent 跑的那个检查**失败了**（seq 12，`exit 1`）；然后它照常收工。这次运行之所以是 `exit 0`，是因为 *agent* 退出了 0。

### 5. 离线重放，想几次几次，不要钱

```console
$ orca replay last
info replay.done reused=2/2 exact=2 divergences=0 unmatched=0 exit=0
```

没有网络、没有 token、没有偏差。想看它按真实的节奏播放一遍，用 `orca replay last --ui`（或 `orca ui`）—— 打开的是一个自包含 HTML 文件，不用起服务，不用装任何东西。

### 6. 想要因果关系而不是时间顺序

```console
$ orca graph last
FROM              TO               KIND      WHY
3 model.response  4 tool.call      recorded  tool_use block in the response
4 tool.call       6 fs.change      inferred  changed path appears in tool input, same or previous turn
4 tool.call       7 tool.result    recorded  tool result answers its call
7 tool.result     8 model.request  recorded  tool_result block in the request
11 shell.exec     12 shell.result  recorded  shell result answers its exec

  1 inferred — derived from this trace, not recorded in it
```

两种边要分开看：**recorded** 是运行当时就写下来的事实（`tool_use` 块物理上就在发出它的那个响应里）；**inferred** 是刚才按它给出的那条规则推出来的 —— 文件系统快照是每轮一次而不是每次工具调用一次，所以把某次文件改动归给某个具体调用，是合理的猜测而不是事实。推断出来的边永远不会被写回 trace，就像 checkpoint 是推导出来的、从不录制一样。

### 7. 问那个你真正想问的问题——换个模型会不会做对

```console
$ orca compare last --from 5 --models claude-opus-5,claude-haiku-4-5 --verify "npm test"
MODEL             VERDICT  TOKENS  COST       WALL  RUN
claude-opus-5     pass     201/25  $0.004890  0.3s  run_1457b35062ba
claude-haiku-4-5  pass     201/25  $0.000326  0.3s  run_b8ee08479fb6
```

两个都过了，一个便宜 **15 倍**。文件相同、对话前缀相同、checkpoint 相同 —— 模型是唯一变了的东西，这也是这个数字能说明问题的唯一原因。

checkpoint 不是录出来的，是推导出来的：对话前缀已经完整、工作区快照也已经就位的那些点。所以每一次 fork 都从一个**确实存在过**的状态出发。

### 8. 把结论交给别人

```bash
orca export last -o bug.html      # 一份自包含的时间线，可以直接贴进 issue
```

## 实用建议

- **别只看 `exit=0`**。上面的例子里，`exit 0` 和 `SHELL ... exit 1` 同时出现。要盯着的是文件树快照（`SNAP` / `FILE`）和每条 shell 的真正退出码。
- **`exact=` 是回归信号**。日常把 `--json` 用来看健康度：`orca replay last --json` 里 `divergences` 不为 0，说明今天的 agent 已经不完全按录像走了。
- **重播免费，比较要花钱**。`orca replay` 永远离线免费；`orca compare` 会真的调模型、真的消耗 token —— 通过 MCP 调工具的 agent 只读得到工具描述那一行，所以 `orca_compare` 在自己的描述里就写明了它会花钱。
- **trace 不要随手外发**。它是 provider 边界上的原始字节，里面是完整的 prompt 和响应。分享之前先用 `orca export ... -o bug.html` 导出受控的时间线，而不是直接贴 trace。
- **录制长期开着没有坏处**：`orca record` 只多一次本地写盘，录制期间 agent 的行为完全不变，`orca doctor` 可以随时确认状态。
- **如果 trace 里只有网关自己的调用**，检查 OpenClaw 是用什么方式拉起子进程的：录制靠的是「子进程继承父进程环境」这条操作系统性质，中间只要有一层把环境清了，它就不再成立。

## 相关链接

- [OrcaReplay 仓库](https://github.com/Continuum-AI-Corp/OrcaReplay)
- [OrcaReplay README · Which agents](https://github.com/Continuum-AI-Corp/OrcaReplay#which-agents) —— 「A gateway that launches the coding agent」一节解释了为什么「网关 + 子进程」这种结构能被一次录全
- [trace 规范 spec/orca-trace-v0.md（CC BY 4.0）](https://github.com/Continuum-AI-Corp/OrcaReplay/blob/main/spec/orca-trace-v0.md)
- 第三方收录：[Awesome Claude Code](https://github.com/hesreallyhim/awesome-claude-code) 已收录 OrcaReplay
