# arXiv 论文阅读与 LaTeX 论文写作

> 含国内适配：中文 LaTeX 配置 / Docker 国内镜像 / 高校论文模板 / 在线平台对比
>
> 📝 本文档合并了上游的 arxiv-paper-reader 和 latex-paper-writing 两个用例，因二者共享 Prismer 工具链且适配内容高度重叠。

读 arXiv 论文意味着下载 PDF、在多篇论文之间频繁切换丢失上下文、艰难解读密集的 LaTeX 公式。写论文意味着安装动辄几个 GB 的 TeX Live、调试编译错误、在编辑器和 PDF 预览之间来回切换。如果你的智能体能在对话中帮你完成论文阅读、分析、写作和编译的全流程呢？

这个用例将两个学术工作流整合到一个智能体中：

- **论文阅读**：通过 arXiv ID 获取论文，自动展平 LaTeX 源码为可读文本，浏览章节结构，批量对比摘要，让智能体做总结和分析
- **论文写作**：在对话中协作撰写 LaTeX 论文，即时编译为 PDF 预览，支持 pdflatex/xelatex/lualatex，无需本地安装 TeX 环境

## 痛点

- **阅读效率低**：arXiv 论文以 PDF 形式发布，难以快速定位关键章节；同时跟踪多篇相关论文时上下文频繁丢失
- **环境配置繁琐**：安装完整的 TeX Live 需要数 GB 空间，不同论文模板的依赖各不相同，编译报错信息晦涩难懂
- **工具切换频繁**：阅读用一个工具、写作用一个编辑器、编译用命令行、预览用 PDF 阅读器——流程断裂

## 它能做什么

| 功能 | 说明 |
|------|------|
| **论文获取** | 通过 arXiv ID 获取论文，自动下载并展平 LaTeX 源码（移除 `\input` 嵌套）为干净的可读文本 |
| **章节浏览** | 先列出论文的章节结构，决定重点阅读哪些部分，而不是通读全文 |
| **摘要速扫** | 批量获取多篇论文的摘要，生成对比表格，按研究方向相关性排序 |
| **智能分析** | 让智能体总结关键贡献、分析方法论、评价实验结果、对比不同论文 |
| **本地缓存** | 已读论文缓存在本地，再次访问瞬间完成 |
| **LaTeX 写作** | 用自然语言描述段落内容，智能体生成对应的 LaTeX 源码 |
| **即时编译** | 支持 pdflatex、xelatex、lualatex 编译，无需本地安装 TeX 环境 |
| **PDF 预览** | 编译后直接在对话中预览 PDF，无需切换应用 |
| **模板支持** | 内置 article、IEEE、beamer、中文文章等起始模板 |
| **参考文献** | 支持 BibTeX/BibLaTeX，粘贴 `.bib` 内容即可自动集成 |

## 所需技能

### 论文阅读

- [arxiv-reader](https://github.com/Prismer-AI/Prismer/tree/main/skills/arxiv-reader) 技能（3 个工具：`arxiv_fetch`、`arxiv_sections`、`arxiv_abstract`）

arxiv-reader 技能无需 Docker 或 Python，使用 Node.js 内置模块运行。它直接从 arXiv 下载 LaTeX 源码，解压并自动展平 `\input` 引用。

### 论文写作

- [latex-compiler](https://github.com/Prismer-AI/Prismer/tree/main/skills/latex-compiler) 技能（4 个工具：`latex_compile`、`latex_preview`、`latex_templates`、`latex_get_template`）
- [Prismer](https://github.com/Prismer-AI/Prismer) 工作区容器（在端口 8080 运行 LaTeX 服务，包含完整的 TeX Live 环境）

## 如何设置

### 1. 安装论文阅读技能

从 Prismer 仓库安装 `arxiv-reader` 技能——将 `skills/arxiv-reader/` 目录复制到你的 OpenClaw 技能文件夹：

```bash
git clone https://github.com/Prismer-AI/Prismer.git
cp -r Prismer/skills/arxiv-reader/ ~/openclaw/skills/
```

技能安装完成后即可直接使用，无需额外配置。

### 2. 部署论文写作环境

克隆并通过 Docker 部署 Prismer（LaTeX 服务与完整 TeX Live 随容器自动启动）：

```bash
git clone https://github.com/Prismer-AI/Prismer.git && cd Prismer
docker compose -f docker/docker-compose.dev.yml up
```

`latex-compiler` 技能为内置技能，无需单独安装。

### 3. 配置论文阅读提示词

向 OpenClaw 发送以下提示词来设定论文阅读工作流：

> 以下提示词建议保留英文使用，效果最佳。中文说明帮助你理解每一步的作用。

```text
I'm researching [topic]. Here's my workflow:

1. When I give you an arXiv ID (like 2301.00001):
   - First fetch the abstract so I can decide if it's relevant
   - If I say "read it", fetch the full paper (remove appendix by default)
   - Summarize the key contributions, methodology, and results

2. When I give you multiple IDs:
   - Fetch all abstracts and give me a comparison table
   - Rank them by relevance to my research topic

3. When I ask about a specific section:
   - List the paper's sections first
   - Then fetch and explain the relevant section in detail

Keep a running list of papers I've read and their key takeaways.
```

试一试：发送 "Read 2401.04088 -- what's the main contribution?" 验证论文阅读功能。

### 4. 配置论文写作提示词

```text
Help me write a research paper in LaTeX. Here's my workflow:

1. Start from the IEEE template (or article/beamer depending on what I need)
2. When I describe a section, generate the LaTeX source for it
3. After each major edit, compile and preview the PDF so I can check formatting
4. If there are compilation errors, read the log and fix them automatically
5. When I provide BibTeX entries, add them to the bibliography and recompile

Use xelatex if I need Chinese/CJK support, otherwise default to pdflatex.
Always run 2 passes for cross-references.
```

试一试：发送 "Start a new IEEE paper titled 'A Survey of LLM Agents'. Give me the template with abstract and introduction sections filled in, then compile it." 验证写作和编译功能。

## 实用建议

- **先阅读再写作**：利用 arxiv-reader 研究相关论文，梳理研究脉络后再开始写作，智能体可以帮你维护一份阅读笔记
- **章节优先于全文**：用 `arxiv_sections` 先浏览结构，精准定位需要深入阅读的章节，避免在不相关内容上浪费时间
- **摘要速扫做文献筛选**：拿到一批候选论文 ID 后，先批量获取摘要做快速筛选，再逐篇深入阅读通过筛选的论文
- **编译错误交给智能体**：LaTeX 编译报错时，不要自己去读日志，让智能体分析报错并自动修复
- **交叉引用需要两次编译**：涉及引用、参考文献、图表编号时，智能体会自动执行两次编译以确保引用正确

## 中国用户适配

国内学术场景与海外有一些不同：中文论文写作需要 xelatex + ctex 宏包支持，Docker 镜像拉取需要配置国内源，高校学位论文有特定模板要求。以下是针对这些场景的适配方案。

### 中文 LaTeX 写作配置

使用 Prismer 的 LaTeX 编译功能撰写中文论文时，需要指定 xelatex 编译器并加载 ctex 宏包。在提示词中告知智能体：

```text
我需要写中文论文。请注意：
1. 编译器使用 xelatex（不是 pdflatex）
2. 文档头部加载 ctex 宏包：\usepackage[UTF8]{ctex}
3. 中文字体使用系统自带方案（fontset=auto 让 ctex 自动选择）
4. 如果需要自定义字体，使用 \setCJKmainfont{} 设置

每次编译都使用 xelatex，并执行两次编译以确保目录和引用正确。
```

一个最小的中文 LaTeX 模板示例：

```latex
\documentclass[12pt,a4paper]{article}
\usepackage[UTF8]{ctex}          % 中文支持核心宏包
\usepackage{geometry}
\geometry{left=2.5cm,right=2.5cm,top=2.5cm,bottom=2.5cm}
\usepackage{amsmath,amssymb}     % 数学公式
\usepackage{graphicx}            % 图片
\usepackage{hyperref}            % 超链接
\usepackage[backend=biber,style=gb7714-2015]{biblatex}  % 中文参考文献格式

\title{论文标题}
\author{作者姓名}
\date{\today}

\begin{document}
\maketitle
\begin{abstract}
摘要内容。
\end{abstract}

\section{引言}
正文内容。

\printbibliography
\end{document}
```

> **注意**：Prismer 的 Docker 容器包含完整的 TeX Live 环境，已内置 ctex 宏包和常用中文字体（Fandol 字体集）。如需使用系统字体（如宋体、黑体），需要在 Docker 容器中挂载字体目录。

### 国内高校学位论文模板

中文学术场景中最常见的需求是学位论文排版。以下是主流高校的 LaTeX 模板，均开源且活跃维护：

| 高校 | 模板名称 | GitHub 地址 | 说明 |
|------|---------|-------------|------|
| 清华大学 | ThuThesis | [tuna/thuthesis](https://github.com/tuna/thuthesis) | 清华大学学位论文模板，由 TUNA 维护 |
| 中国科学院 | ucasthesis | [mohuangrui/ucasthesis](https://github.com/mohuangrui/ucasthesis) | 国科大学位论文模板 |
| 浙江大学 | zjuthesis | [TheNetAdmin/zjuthesis](https://github.com/TheNetAdmin/zjuthesis) | 支持本硕博，Overleaf 可用 |
| 中国科技大学 | ustcthesis | [ustctug/ustcthesis](https://github.com/ustctug/ustcthesis) | 中科大学位论文模板 |
| 更多模板 | latex-templates | [hantang/latex-templates](https://github.com/hantang/latex-templates) | 国内高校 LaTeX 模板大全 |

**使用方式**：下载模板后，可以让 OpenClaw 智能体基于模板结构帮你填充内容并编译：

```text
我要写硕士毕业论文，使用清华大学 ThuThesis 模板。
模板文件已在 ~/thesis/ 目录下。请：
1. 阅读模板结构，了解各文件的作用
2. 根据我的描述生成各章节内容
3. 编译并预览，确保格式符合模板要求
```

### Docker 国内镜像配置

Prismer 的 Docker 镜像包含完整的 TeX Live 环境，镜像体积较大。国内用户拉取镜像时建议配置 Docker 镜像加速：

配置 Docker 镜像加速器（国内 Docker Hub 镜像站变化较快，请搜索“Docker 国内镜像加速”获取最新可用地址）：

编辑 `/etc/docker/daemon.json`（不存在则新建）：

```json
{
  "registry-mirrors": [
    "https://your-mirror.example.com"
  ]
}
```

重启 Docker 服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**TeX Live 包管理器 (tlmgr) 换源**：

如果需要在容器内安装额外的 LaTeX 宏包，可以将 CTAN 源切换到清华镜像：

```bash
tlmgr option repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
```

> 💡 也可使用阿里云容器镜像服务（需注册账号获取专属加速地址）。
>
> **macOS 用户**：macOS 版 Docker Desktop 的镜像加速在 Docker Desktop > Settings > Docker Engine 中配置，添加相同的 `registry-mirrors` 字段即可。

### 国内论文数据库说明

本用例的论文阅读功能依赖 arXiv 的公开 API，以下是国内学术数据库的兼容性说明：

| 数据库 | 能否使用 | 说明 |
|--------|:-------:|------|
| **arXiv** | 可用 | 提供免费公开 API，无需注册，AI/ML/CS 领域论文覆盖全面 |
| **Semantic Scholar** | 可用 | 提供免费 API，可作为论文发现的补充来源 |
| **知网 (CNKI)** | 不可用 | 未提供公开 API，内容访问需要机构授权 |
| **万方数据** | 不可用 | 未提供公开 API，需通过高校图书馆等机构访问 |
| **维普** | 不可用 | 未提供公开 API |

**实际影响**：对于 AI、机器学习、计算机科学领域的研究者，arXiv 已覆盖绝大多数重要论文。中文论文（如发表在知网、万方的论文）目前无法通过本用例的自动化流程处理，需要手动下载后使用 OpenClaw 的文件读取功能分析。

### 国内 LaTeX 在线平台对比

如果不想使用 Docker 部署本地 LaTeX 环境，以下是国内可用的在线 LaTeX 编辑平台：

> 以下为国内可用的在线 LaTeX 平台，仅供参考，非推荐。请根据自身需求选择。

| 平台 | 特点 | 费用 | 适用场景 |
|------|------|------|---------|
| [Overleaf](https://www.overleaf.com) 国际版 | 功能最全面，模板生态最丰富 | 免费版有编译时长限制；付费版（价格见 [Overleaf 官网](https://www.overleaf.com/user/subscription/plans)） | 国际期刊投稿、与海外合作者协作 |
| [TeXPage](https://www.texpage.com) | 国内团队开发，端到端加密，服务器在国内 | 付费 | 对数据安全和网络延迟敏感的场景 |
| [LoongTeX](https://www.loongtex.com) | 国产平台，AI 辅助校对，支持异步协作 | 提供免费版 | 国内高校团队协作 |

**与 Prismer 方案的对比**：在线平台的优势是零配置，但不支持通过 OpenClaw 智能体自动化写作和编译流程。Prismer + OpenClaw 的组合适合需要在对话中完成"阅读相关论文 -> 写作 -> 编译 -> 修改"全流程自动化的用户。

### 适用人群

- **AI/ML 研究者**：每天需要阅读 arXiv 新论文，用英文撰写期刊/会议论文
- **国际期刊投稿者**：需要使用 IEEE、ACM 等特定 LaTeX 模板
- **研究生**：需要用高校指定模板撰写学位论文
- **跨语言写作者**：中英文混排论文（如中文学位论文中引用英文文献）

## 相关链接

- [Prismer - 开源研究平台](https://github.com/Prismer-AI/Prismer) -- 提供 arxiv-reader 和 latex-compiler 技能
- [arxiv-reader 技能](https://github.com/Prismer-AI/Prismer/tree/main/skills/arxiv-reader) -- 论文阅读工具（3 个工具）
- [latex-compiler 技能](https://github.com/Prismer-AI/Prismer/tree/main/skills/latex-compiler) -- LaTeX 编译工具（4 个工具）
- [arXiv API 文档](https://info.arxiv.org/help/api/) -- arXiv 官方 API
- [ctex 宏包文档](https://ctan.org/pkg/ctex) -- 中文 LaTeX 排版核心宏包
- [ThuThesis - 清华大学论文模板](https://github.com/tuna/thuthesis)
- [国内高校 LaTeX 模板合集](https://github.com/hantang/latex-templates) -- 收录各高校学位论文模板
- [清华 CTAN 镜像](https://mirrors.tuna.tsinghua.edu.cn/CTAN/) -- TeX Live 国内镜像源
- [TeXPage](https://www.texpage.com) -- 国内 LaTeX 在线编辑平台
- [LoongTeX](https://www.loongtex.com) -- 国产 LaTeX 协作平台

---

**原文链接**：
- [arXiv Paper Reader (English)](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/53)
- [LaTeX Paper Writing (English)](https://github.com/hesamsheikh/awesome-openclaw-usecases/pull/53)
