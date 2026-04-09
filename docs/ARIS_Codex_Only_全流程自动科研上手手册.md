# ARIS 纯 Codex 全流程自动科研上手手册

> 本文档面向当前只有 Codex 可用、Claude 账号暂时不可用的情况。
>
> 目标不是解释 ARIS 的所有分支与历史，而是专门回答一个更实际的问题：
>
> **如果你现在只能使用 Codex，那么怎样从零开始，用纯 Codex 跑通一次 ARIS 的自动科研全流程，并借此理解 ARIS 的使用方式？**

## 0. 文档定位

这份手册是对你现有主文档的补充，专门讲：

- 纯 Codex 路径怎么安装
- 不依赖 Claude Code 时，ARIS 由谁执行、skill 怎么被调用
- 从新建项目到 idea discovery、实验、review、写论文的全流程怎么走
- 你作为小白第一次应该怎么一步一步试

它默认你已经有这些前提：

- 你已经 fork / clone 并整理好了 `~/ARIS`
- 你已经有项目服务器和 GitHub 仓库
- 你的相关配置过程可参考：
  [`/data_3/wly/ARIS/personal/docs/ARIS_Setting.md`](/data_3/wly/ARIS/personal/docs/ARIS_Setting.md)

## 1. 先说结论：纯 Codex 路径到底是什么

纯 Codex 路径对应 ARIS README 里的 **方案 F**：

- **执行者**：Codex CLI
- **审稿者**：Codex 自己拉起的次级 agent（`spawn_agent`）
- **不依赖 Claude Code**
- **需要 OpenAI / Codex 这一条链可用**

这条路径的核心不是：

- Claude Code 读取 `skills/`

而是：

- Codex CLI 读取 `~/.codex/skills/`
- 其中安装的是 `skills/skills-codex/*`
- review-heavy 步骤不走 Claude MCP，而是由 Codex 在会话内部通过 `spawn_agent` / `send_input` 拉起次级 reviewer agent

也就是说，纯 Codex 版 ARIS 的结构是：

```text
你
→ Codex CLI（主执行者）
→ ~/.codex/skills/* 中的 ARIS Codex 技能
→ 主 agent 读 skill 并执行
→ review-heavy 环节用 spawn_agent 拉起次级 Codex reviewer
→ 结果写回项目文件
```

## 2. 纯 Codex 路径和 Claude 路径有什么本质差异

你前一份总手册里的主路径是：

```text
Claude Code 执行
→ ~/.claude/skills/
→ 审稿常通过 Codex MCP
```

而你现在要走的是：

```text
Codex CLI 执行
→ ~/.codex/skills/
→ 审稿由 Codex 的次级 agent 完成
```

最重要的差异有 6 个。

### 2.1 宿主变了

现在的宿主不再是 Claude Code，而是 Codex CLI。

你启动研究项目时，通常不再输入：

```bash
claude
```

而是输入：

```bash
codex -C /你的项目目录
```

### 2.2 技能目录变了

现在不安装到：

```text
~/.claude/skills/
```

而是安装到：

```text
~/.codex/skills/
```

### 2.3 技能包也变了

不是安装主线 `skills/*`，而是安装专门的 Codex 镜像包：

```text
skills/skills-codex/*
```

这个包的作用是：

- 保留主线 ARIS 的同名科研技能
- 把控制型工作流适配为 Codex 能执行的形式
- 在 review-heavy 步骤里改用 `spawn_agent` / `send_input`

### 2.4 项目配置文件重点从 `CLAUDE.md` 变成 `AGENTS.md`

在纯 Codex 路径里，许多关键 skill 明确写的是去读项目中的 `AGENTS.md`。

尤其是：

- `run-experiment`
- `experiment-bridge`
- `research-lit`

例如 Codex 版 `run-experiment` 会从 `AGENTS.md` 读取：

- 远程服务器 SSH
- conda 激活命令
- 远程代码目录
- `code_sync` 方式
- W&B 配置

因此对纯 Codex 路径来说，**项目里的 `AGENTS.md` 是核心配置文件**。

### 2.5 reviewer 的实现方式变了

在主线路径里，reviewer 常是外部 MCP 或外部模型。

在纯 Codex 路径里，很多 review-heavy skill 明确写成：

- `spawn_agent`
- `send_input`

例如 Codex 版 `auto-review-loop` 明确写着：

- Round 1 用 `spawn_agent`
- Round 2+ 用 `send_input`
- reviewer model 默认是 `gpt-5.4`

也就是说，纯 Codex 路径不是“没有 reviewer”，而是“reviewer 内置为 Codex 的次级 agent”。

### 2.6 Session 恢复提示文件也更偏向 `codex.md`

根据仓库的 session recovery 文档，Codex CLI 的建议做法是：

- 把状态读取说明写进 system prompt
- 或放进 `codex.md`

但不管你是否真的使用 `codex.md`，**项目级状态文件**仍然是核心。

## 3. 你现在这台机器对纯 Codex 路径意味着什么

我已经核对过你当前服务器上的 Codex 状态，结论很明确：

- `codex` 命令已经安装可用
- `~/.codex/config.toml` 已存在
- `~/.codex/skills/` 里目前还没有安装 ARIS 的 Codex 技能包
- `codex mcp list` 当前为空

这意味着：

1. 你已经具备 Codex CLI 基础运行能力。
2. 你还没有把 ARIS 的纯 Codex 技能真正装进去。
3. 对于“纯 Codex 基础路径”而言，**先装 `skills/skills-codex/*` 才是第一步**。
4. 当前没有 MCP server 不一定是问题，因为纯 Codex 基础路径的 reviewer 主要依赖次级 agent，而不是额外 MCP。

## 4. 纯 Codex 路径下，ARIS 的完整系统结构

你可以把它理解成下面这张图：

```text
ARIS 主线仓库（提供 Codex 技能源）
→ skills/skills-codex/*
→ 复制到 ~/.codex/skills/
→ 你在研究项目目录中运行 codex -C <project>
→ Codex 读取 AGENTS.md / 项目文件 / skill
→ 主 agent 执行文献、实现、实验、写作
→ review-heavy 步骤用 spawn_agent 拉次级 reviewer
→ 输出写入 IDEA_REPORT.md / EXPERIMENT_LOG.md / AUTO_REVIEW.md / paper/ ...
```

这套体系里，最重要的 5 个层次分别是：

### 4.1 技能源

来源是：

```text
/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/
```

### 4.2 运行时技能目录

Codex 真正读取的是：

```text
~/.codex/skills/
```

### 4.3 项目配置层

纯 Codex 路径下，你最应该维护的是：

- `AGENTS.md`
- 可选 `codex.md`
- 研究输入文件，如 `RESEARCH_BRIEF.md`

### 4.4 项目状态层

随着流程推进，项目里会不断生成：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- `NARRATIVE_REPORT.md`
- `paper/`

### 4.5 子 agent 审稿层

纯 Codex 路径里最关键的 review-heavy skill，通常会让主 agent：

1. 先整理上下文
2. 再 `spawn_agent`
3. 让次级 agent 用更高推理强度做 reviewer
4. 主 agent 再根据 reviewer 结果改代码、补实验、改叙事

这就是纯 Codex 版的“执行者 / 审稿者分离”。

## 5. 纯 Codex 路径下需要安装什么

先给你一个最小清单。

### 5.1 必需项

| 组件 | 是否需要 | 作用 |
|---|---|---|
| Codex CLI | 必需 | 宿主执行器 |
| OpenAI / Codex 可用额度 | 必需 | 纯 Codex 路径的模型调用来源 |
| `~/.codex/skills/` | 必需 | 存放 ARIS Codex 技能 |
| Git | 必需 | 管理项目与同步代码 |
| Python 3 | 强烈建议 | 跑辅助脚本、论文抓取、wiki 等 |

### 5.2 视功能而定的可选项

| 组件 | 何时需要 |
|---|---|
| LaTeX (`texlive-full`, `latexmk`) | 需要编译论文 PDF 时 |
| W&B | 想让实验训练过程可视化和自动健康检查时 |
| Zotero MCP / Obsidian MCP | 想把个人论文库 / 笔记库接入 `research-lit` 时 |
| Gemini / Claude 审稿 overlay | 你以后想让 reviewer 不再是 Codex 次级 agent 时 |

## 6. 你现在应该采用哪条纯 Codex 上手路线

对于你当前这个阶段，最适合的不是一上来就：

```text
/research-pipeline "课题"
```

而是：

1. 安装 `skills/skills-codex/*`
2. 在独立项目目录里写好 `AGENTS.md`
3. 先体验一次分步流程
4. 理解每一步产物
5. 再尝试全自动总控

原因很简单：

- 你目前还在学习 ARIS 本身
- 你当前又只有 Codex 一条宿主路径
- 分步流程更容易看清“skill 做了什么、文件怎么流转”

后面这份手册会按这个学习顺序展开。

## 7. 第一次使用前，你应该怎样把纯 Codex 路径装好

这一节是你真正要操作的准备步骤。

### 7.1 第一步：确认 Codex CLI 本身可用

先在终端中检查：

```bash
codex --help
```

如果这条命令能正常返回帮助信息，说明 Codex CLI 已安装。

根据我刚才在你这台机器上的检查：

- `codex` 命令可用
- `~/.codex/config.toml` 已存在

所以你不需要再从零装 Codex CLI，本机这一项已经具备。

另外你要注意一个版本差异：

- ARIS 仓库里有些旧说明会写 `codex setup`
- 但你当前这台机器上的 Codex CLI 帮助里显示的是 `codex login`

也就是说，对你现在这个环境，应当优先以本机实际命令为准。

如果未来你在别的机器上从零配置 Codex，更稳妥的方式是先看：

```bash
codex --help
codex login --help
```

再按当前 CLI 版本执行，而不是机械照搬旧文档里的 `codex setup`。

### 7.2 第二步：把 ARIS 的 Codex 技能安装到 `~/.codex/skills/`

这是你当前最缺的一步。

执行：

```bash
cd /data_3/wly/ARIS/Auto-claude-code-research-in-sleep
mkdir -p ~/.codex/skills
cp -a skills/skills-codex/* ~/.codex/skills/
```

这一步完成后，Codex 才能在自己的技能目录里发现 ARIS 的纯 Codex 技能。

你要理解成：

- 仓库里的 `skills/skills-codex/` 是技能源代码
- `~/.codex/skills/` 是 Codex 真正运行时会读的技能目录

### 7.3 第三步：理解“纯 Codex 基础路径默认不要求额外 reviewer MCP”

你现在最需要的是**基础纯 Codex 路径**。

这条路径的 reviewer 主要依赖：

- `spawn_agent`
- `send_input`

因此你不需要像 Codex+Claude、Codex+Gemini 那样，先去安装 reviewer overlay 和本地 bridge，才能开始第一次体验。

也就是说，**第一次上手纯 Codex 基础版时，可以不配置额外 MCP reviewer**。

### 7.4 第四步：是否需要配置 MCP

分两种情况。

#### 必须马上配的 MCP

纯 Codex 基础版第一次体验里，通常没有“必须马上配置”的 reviewer MCP。

#### 可选 MCP

如果你想增强文献检索或知识库能力，后续可以再加：

- Zotero MCP
- Obsidian MCP

但它们不是第一次跑通主流程的必要条件。

### 7.5 第五步：推荐安装的其他工具

#### LaTeX

如果你要让 `paper-writing` 真正编译出 PDF，建议安装：

```bash
sudo apt install texlive-full latexmk poppler-utils
```

#### Python 科研环境

如果你的实验涉及训练脚本，项目本身还需要：

- conda 或 uv
- PyTorch
- CUDA
- 你研究代码自己的依赖

ARIS 不替你装这些；它只是组织并驱动研究流程。

### 7.6 第六步：关于 Codex 的启动参数，你应该怎么理解

`codex --help` 里最相关的几个选项是：

- `-C, --cd <DIR>`：指定项目目录
- `--search`：允许实时 web search
- `-s, --sandbox`：设置命令执行沙箱
- `-a, --ask-for-approval`：设置审批策略

对于 ARIS 的科研流程，最相关的是：

#### `-C /path/to/project`

告诉 Codex 以哪个项目目录为工作根目录。

#### `--search`

非常重要。因为 `research-lit`、`novelty-check` 等工作流往往需要联网搜索。

#### `-s workspace-write`

适合第一次体验。允许在项目内写文件，但保持相对安全。

#### `-a on-request`

适合第一次体验。让 Codex 在需要时请求批准，而不是完全裸奔。

所以，第一次体验时，我更推荐你这样启动：

```bash
codex -C /你的项目目录 --search -s workspace-write -a on-request
```

如果你以后非常熟悉流程，并且运行环境本身已经由你外部隔离好了，才考虑更激进的自动化模式。

## 8. 纯 Codex 项目应该如何初始化

这一节是你真正开始一个研究项目时要做的事情。

### 8.1 不要在 ARIS 仓库里直接做研究

和主线路径一样，纯 Codex 路径下也不应该直接在：

```text
/data_3/wly/ARIS/Auto-claude-code-research-in-sleep
```

里做你的研究。

正确做法是另外建一个研究项目目录，例如：

```bash
mkdir -p /data_3/wly/research/ddlm-gap
cd /data_3/wly/research/ddlm-gap
mkdir -p docs refine-logs results paper papers
```

### 8.2 纯 Codex 路径里最重要的项目配置文件：`AGENTS.md`

这是纯 Codex 版的核心。

很多 Codex 版 skill 都显式要求去读项目里的 `AGENTS.md`。

尤其是：

- `run-experiment`
- `experiment-bridge`
- `research-lit`

所以你第一次开项目时，最应该先写的是：

```text
AGENTS.md
```

### 8.3 `AGENTS.md` 里应该写什么

至少建议写 5 类信息：

1. 研究目标
2. 远程服务器配置
3. 本地环境配置
4. 代码同步策略
5. 当前流程状态

### 8.4 一个适合你当前情况的 `AGENTS.md` 示例

下面给你一份更适合纯 Codex + 远程服务器 + GitHub 已配置好的写法。

```markdown
# Project Agents Guide

## Project Goal
- 研究离散扩散语言模型中的 factorized gap。
- 目标是形成一篇可投稿论文，而不是只做工程复现。

## Constraints
- 总预算：200 GPU-hours
- 时间窗口：4 个月
- 优先做中等规模验证，不做超大规模预训练

## Remote Server
- SSH: `ssh my-gpu-server`
- GPU: 4x A100 80GB
- Conda: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- Code dir: `/home/wly/projects/ddlm-gap`
- code_sync: git
- wandb: false

## Local Environment
- Python env: `conda activate research`
- Root code dir: `/data_3/wly/research/ddlm-gap`

## Paper Library
- Local papers path: `papers/`

## Working Rules
- 所有实验结果必须写入 `results/`
- 所有关键实验要追加到 `EXPERIMENT_LOG.md`
- 所有 claim 更新要同步到 `docs/research_contract.md`
- 所有重要表格优先保存为 JSON 或 CSV

## Pipeline Status
- Current stage: project initialization
- Current active idea: none
- Next recommended step: /idea-discovery
```

### 8.5 为什么这里推荐 `code_sync: git`

你前面已经告诉我：

- 你的服务器端和 GitHub 仓库已经配置好
- 相关过程在 `ARIS_Setting.md` 中已有记录

因此对你来说，`code_sync: git` 是很合理的选择。

它的优点是：

- 远程和本地代码状态更清晰
- 变更有 commit 记录
- 多次实验部署更稳定

但你必须注意一个前提：

**如果你写的是 `code_sync: git`，那么这个研究项目本身就应该是一个 Git 仓库。**

也就是说，在研究项目目录里你至少应该完成：

```bash
cd /data_3/wly/research/ddlm-gap
git init
git remote add origin <你的研究项目远程仓库地址>
git add .
git commit -m "init research project"
git push -u origin main
```

并且在远程服务器的 `Code dir` 中，应该已经有这个仓库的 clone。

如果你暂时不想把这个研究项目单独做成 Git 仓库，就不要写 `code_sync: git`，而改用：

```text
code_sync: rsync
```

当然，如果你的研究项目不是一个独立 Git 仓库，而只是一个本地目录，也可以改成：

```text
code_sync: rsync
```

### 8.6 是否还需要 `RESEARCH_BRIEF.md`

强烈建议要。

纯 Codex 路径也同样受益于详细研究输入。你可以在项目根目录写：

```text
RESEARCH_BRIEF.md
```

建议内容包括：

- 问题定义
- 已做尝试
- 失败经验
- 计算预算
- 目标会议
- 非目标

这会显著提高 `idea-discovery` 的质量。

### 8.7 一个重要边界：纯 Codex 基础包里没有 `research-wiki`

你前一份总手册里提到的 `research-wiki`，在纯 Codex 基础技能包 `skills/skills-codex/` 里**并不在默认集合中**。

这意味着：

- 第一次纯 Codex 上手时，不要把 `research-wiki` 当成必要步骤
- 你的主流程应聚焦于：
  - `idea-discovery`
  - `experiment-bridge`
  - `monitor-experiment`
  - `result-to-claim`
  - `auto-review-loop`
  - `paper-writing`

也就是说，纯 Codex 基础版完全可以跑通一次科研闭环，只是默认不包含长期 wiki 记忆这条线。

### 8.8 是否还需要 `codex.md`

不是必需，但建议有。

根据仓库的 session recovery 文档，Codex CLI 的建议之一是把状态读取说明写进：

- system prompt
- 或 `codex.md`

对你来说，一个实用写法是让 `codex.md` 只做“会话恢复入口”，例如：

```markdown
# Codex Session Rules

- 每次进入新 session，先阅读：
  - AGENTS.md
  - RESEARCH_BRIEF.md
  - docs/research_contract.md（如果存在）
  - IDEA_REPORT.md（如果存在）
  - EXPERIMENT_LOG.md（如果存在）
  - AUTO_REVIEW.md（如果存在）
  - REVIEW_STATE.json（如果存在）
- 优先依据 `AGENTS.md` 中的 `## Pipeline Status` 判断当前阶段
- 不要跳过阶段性输出文件
```

它的作用不是替代 skill，而是让新 session 更稳定地恢复上下文。

## 9. 纯 Codex 路径下，你在交互时到底怎么发指令

这也是初学者最容易糊涂的地方。

### 9.1 启动会话命令

假设你的项目目录是：

```text
/data_3/wly/research/ddlm-gap
```

那么推荐启动方式是：

```bash
codex -C /data_3/wly/research/ddlm-gap --search -s workspace-write -a on-request
```

### 9.2 进入 Codex 之后，怎么触发 skill

在纯 Codex 路径里，你仍然按 ARIS 的 skill 名称来调用。

典型写法就是：

```text
/idea-discovery "离散扩散语言模型中的 factorized gap"
/experiment-bridge
/auto-review-loop "离散扩散语言模型中的 factorized gap"
/paper-writing "NARRATIVE_REPORT.md"
/research-pipeline "离散扩散语言模型中的 factorized gap"
```

也就是说，从“使用者视角”看，**纯 Codex 路径的 skill 调用方式和主线几乎一致**。

### 9.3 如果 skill 没有像你预期那样被触发怎么办

第一次使用时，可能存在以下情况：

- 技能还没装到 `~/.codex/skills/`
- 当前会话没有正确读取技能目录
- 你想让 Codex 先理解项目再跑技能

这种情况下，你可以先发一个“预热指令”：

```text
请先读取当前项目中的 AGENTS.md、RESEARCH_BRIEF.md，并检查可用 skills。确认可以使用 /idea-discovery、/experiment-bridge、/auto-review-loop、/paper-writing。
```

然后再正式开始。

### 9.4 你输入 slash 指令后，Codex 内部会怎么做

例如你输入：

```text
/idea-discovery "离散扩散语言模型中的 factorized gap"
```

内部典型流程是：

1. Codex 识别到你要调用一个 skill。
2. 在 `~/.codex/skills/` 中找到对应目录和 `SKILL.md`。
3. 读取 skill 目标、工作流和输出约定。
4. 读取项目里的 `AGENTS.md`、`RESEARCH_BRIEF.md`、现有结果文件。
5. 执行 skill 内部定义的工作流。
6. 在需要外部审稿的位置，用 `spawn_agent` 拉起次级 Codex reviewer。
7. 把结果写回项目文件。

这就是为什么你看起来只输入了一句命令，但实际上会触发很长的一条流程链。

## 10. 纯 Codex 路径下，从初始指令到项目完成的全流程到底怎么跑

这一节回答你的第一个核心要求：

**只用 Codex 的情况下，整个系统从初始指令到项目完成，完整链路是什么？**

### 10.1 全流程总图

纯 Codex 路径下，一次完整自动科研通常是：

```text
项目初始化
→ /idea-discovery
→ docs/research_contract.md
→ /experiment-bridge
→ /monitor-experiment
→ /result-to-claim
→ /auto-review-loop
→ NARRATIVE_REPORT.md
→ /paper-writing
→ /paper-slides / /paper-poster（可选）
→ /rebuttal（可选）
```

如果你想一把串起来，则是：

```text
/research-pipeline "你的研究方向"
```

但对第一次体验来说，我还是建议分步。

### 10.2 第 0 阶段：项目初始化

你先完成：

1. 创建项目目录
2. 写 `AGENTS.md`
3. 写 `RESEARCH_BRIEF.md`
4. 可选写 `codex.md`
5. 启动 Codex 会话

这一步的作用是给 Codex 提供：

- 研究目标
- 环境约束
- 服务器信息
- 当前流程状态

如果这一步没做，后面的技能仍然可能工作，但会明显更不稳定。

### 10.3 第 1 阶段：`/idea-discovery`

你输入：

```text
/idea-discovery "你的研究方向"
```

例如：

```text
/idea-discovery "离散扩散语言模型中的 factorized gap" — AUTO_PROCEED: false, arxiv download: true
```

#### 这一步内部会发生什么

Codex 版 `idea-discovery` 的逻辑是：

```text
/research-lit
→ /idea-creator
→ /novelty-check
→ /research-review
→ /research-refine-pipeline
```

更细一点说：

1. `research-lit` 先做文献调研。
2. `idea-creator` 产生 8-12 个候选 idea。
3. `novelty-check` 验证查新，排除撞车方案。
4. `research-review` 拉起次级 Codex reviewer，从高级审稿人角度挑刺。
5. `research-refine-pipeline` 把最有希望的方案打磨成：
   - 可执行的方法提案
   - claim-driven 实验计划

#### 这一步会写什么文件

最关键的输出是：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`

如果你打开了 compact 模式，还会有：

- `IDEA_CANDIDATES.md`

#### 这些文件为什么重要

- `IDEA_REPORT.md`：记录候选 idea、pilot 信号、novelty 结果、reviewer 反馈
- `FINAL_PROPOSAL.md`：记录最终选择方案的核心方法
- `EXPERIMENT_PLAN.md`：记录要跑什么实验、顺序和预算
- `EXPERIMENT_TRACKER.md`：记录每个实验块当前状态

#### 后面的组件如何用这些文件

- `experiment-bridge` 读 `EXPERIMENT_PLAN.md` 和 `FINAL_PROPOSAL.md`
- `run-experiment` 按计划真正部署实验
- `auto-review-loop` 读实验日志和结果继续迭代

### 10.4 第 2 阶段：固化当前选中方案

虽然 skill 产物已经足够多，但为了提高后续 session 稳定性，建议你立即再做一步：

- 创建 `docs/research_contract.md`

你可以手动让 Codex 做：

```text
请根据 IDEA_REPORT.md 和 refine-logs/FINAL_PROPOSAL.md 创建 docs/research_contract.md，只保留当前选中的 idea、核心 claim、实验设计和当前状态。
```

#### 这一步为什么必须强烈建议做

因为 `IDEA_REPORT.md` 通常会非常长，包含很多已经被淘汰或备用的方案。后面恢复 session 时如果总把整个 `IDEA_REPORT.md` 重新塞进上下文，会污染工作记忆。

`docs/research_contract.md` 的作用是：

- 将“当前活跃方案”抽离成一页纸
- 给 Codex 一个更稳定、更聚焦的工作状态文件

### 10.5 第 3 阶段：`/experiment-bridge`

你输入：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md"
```

或者加参数：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — base repo: https://github.com/your/repo, compact: true
```

#### 这一步内部会发生什么

Codex 版 `experiment-bridge` 会做这些事：

1. 读取 `EXPERIMENT_PLAN.md`
2. 读取 `EXPERIMENT_TRACKER.md`
3. 读取 `FINAL_PROPOSAL.md`
4. 扫描项目现有代码，决定哪些要复用、哪些要新增
5. 先实现 sanity-stage 实验
6. 再实现 baseline / main method / ablation 所需脚本
7. 调用 `/run-experiment` 部署实验
8. 收集初始结果
9. 更新 tracker 和结果摘要

#### 这一步会写什么

- 项目代码文件
- 配置文件
- 训练 / 评测脚本
- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `results/` 目录下的各类结果

#### 日志和结果往哪写

最重要的落点是：

- `results/`：原始实验结果
- `EXPERIMENT_LOG.md`：实验摘要与复现实验命令
- `refine-logs/EXPERIMENT_TRACKER.md`：实验状态看板

### 10.6 第 4 阶段：`/run-experiment`

虽然 `experiment-bridge` 会自动用到它，但你也可以单独理解它。

它的职责是：

- 读取 `AGENTS.md`
- 判断本地还是远程环境
- 检查 GPU 是否可用
- 用 `rsync` 或 `git` 同步代码
- 在远程服务器上用 `screen` 启动实验
- 保存日志

#### 纯 Codex 路径下它最依赖什么

最依赖：

- `AGENTS.md` 中的 `## Remote Server`
- `AGENTS.md` 中的 `code_sync`

如果这些没写清楚，它就无法稳定部署实验。

#### 典型日志位置

根据 skill 设计，实验命令通常会通过：

```bash
... 2>&1 | tee <log_file>
```

保存日志，所以：

- 实验脚本运行输出会被保存成日志文件
- `EXPERIMENT_LOG.md` 会保存摘要和复现命令

### 10.7 第 5 阶段：`/monitor-experiment`

你输入：

```text
/monitor-experiment
```

这一步的职责是：

- 看远程 `screen` 是否仍在运行
- 看日志有没有报错
- 看结果文件是否已经生成
- 更新 `EXPERIMENT_TRACKER.md`
- 必要时把完成的实验写入 `EXPERIMENT_LOG.md`

### 10.8 第 6 阶段：`/result-to-claim`

你输入：

```text
/result-to-claim
```

这一步非常关键，因为它负责把“实验数字”翻译成“能不能支撑论文 claim”。

它主要读取：

- `EXPERIMENT_LOG.md`
- `EXPERIMENT_TRACKER.md`
- 当前方法说明

它可能输出或更新：

- claim verdict
- `findings.md`
- `research-wiki/` 中的 claim 支持状态（如果 wiki 已启用）

### 10.9 第 7 阶段：`/auto-review-loop`

你输入：

```text
/auto-review-loop "课题主题" — human checkpoint: true
```

这一步是纯 Codex 路径下最有代表性的技能之一。

#### 内部核心机制

1. 主 Codex agent 读取项目当前上下文。
2. 通过 `spawn_agent` 拉起次级 reviewer agent。
3. reviewer agent 给分、列剩余问题、提出最小修复动作。
4. 主 agent 去实施修复。
5. 主 agent 更新日志与状态。
6. 下一轮继续用 `send_input` 延续 reviewer 线程。

#### 这一步最重要的状态文件

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`

前者记录每一轮完整评审历史，后者记录循环是否完成、停在哪一轮、上次分数多少。

#### 这一步会写什么

- reviewer 原始回复
- 每轮行动列表
- 修复说明
- 结果更新
- 最终方法摘要
- 可选的 `CLAIMS_FROM_RESULTS.md`

### 10.10 第 8 阶段：生成 `NARRATIVE_REPORT.md`

当 review loop 基本完成后，你最好再做一个承上启下的文档整理：

```text
请根据 docs/research_contract.md、EXPERIMENT_LOG.md、AUTO_REVIEW.md 和当前结果，生成 NARRATIVE_REPORT.md。
```

这个文件是工作流 3 的最佳输入。

它汇总：

- 论文核心故事
- 主要 claim
- 实验 setup
- 主结果与解释
- figure / table 设计
- 已知弱点

### 10.11 第 9 阶段：`/paper-writing`

你输入：

```text
/paper-writing "NARRATIVE_REPORT.md" — venue: NeurIPS
```

Codex 版 `paper-writing` 会按顺序调用：

```text
/paper-plan
→ /paper-figure
→ /paper-write
→ /paper-compile
→ /auto-paper-improvement-loop
```

#### 这一步会写什么

- `PAPER_PLAN.md`
- `paper/figures/`
- `paper/main.tex`
- `paper/sections/*.tex`
- `paper/references.bib`
- `paper/main.pdf`
- `paper/PAPER_IMPROVEMENT_LOG.md`

#### 后面的组件如何用

- `paper-slides` 可以读 `paper/`
- `paper-poster` 可以读 `paper/`
- `rebuttal` 可以读论文和 review comment

### 10.12 第 10 阶段：项目完成后的衍生产出

如果你需要展示材料，还可以继续：

```text
/paper-slides "paper/"
/paper-poster "paper/"
```

如果已经收到投稿 review：

```text
/rebuttal "paper/ + reviews" — venue: NeurIPS
```

## 11. 纯 Codex 路径下，一次完整使用范例：模拟一项自动科研从头到尾怎么走

这一节回答你的第二个核心要求：

**给你一个纯 Codex 的完整模拟案例，把每一步命令、skill、结果文件和系统行为都讲清楚。**

### 11.1 场景设定

我们继续用同一个研究主题：

```text
离散扩散语言模型中的 factorized gap
```

项目目录设为：

```text
/data_3/wly/research/ddlm-gap
```

### 11.2 准备阶段：终端命令

先在普通 shell 中执行：

```bash
mkdir -p /data_3/wly/research/ddlm-gap
cd /data_3/wly/research/ddlm-gap
mkdir -p docs refine-logs results paper papers
```

然后确保 Codex 技能已安装：

```bash
cd /data_3/wly/ARIS/Auto-claude-code-research-in-sleep
mkdir -p ~/.codex/skills
cp -a skills/skills-codex/* ~/.codex/skills/
```

### 11.3 写 `AGENTS.md`

在项目目录中写：

```markdown
# Project Agents Guide

## Project Goal
- 研究离散扩散语言模型中的 factorized gap。
- 目标是得到可投稿的方法型论文。

## Constraints
- 总预算：200 GPU-hours
- 截止时间：2026-08-15

## Remote Server
- SSH: `ssh my-gpu-server`
- GPU: 4x A100 80GB
- Conda: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- Code dir: `/home/wly/projects/ddlm-gap`
- code_sync: git
- wandb: false

## Local Environment
- Python env: `conda activate research`
- Root code dir: `/data_3/wly/research/ddlm-gap`

## Paper Library
- Local papers path: `papers/`

## Working Rules
- 所有实验结果写入 `results/`
- 所有实验摘要写入 `EXPERIMENT_LOG.md`
- 所有重要决定写回 `docs/research_contract.md`

## Pipeline Status
- Current stage: initialization
- Current active idea: none
- Next recommended step: /idea-discovery
```

### 11.4 写 `RESEARCH_BRIEF.md`

例如：

```markdown
# Research Brief

## Problem Statement
当前离散扩散语言模型的训练目标与采样轨迹之间存在 factorized gap。
我们怀疑这种错位导致长序列质量退化。

## Background
- Field: NLP
- Sub-area: discrete diffusion LMs
- Key papers I've read: SEDD, D3PM, masked diffusion LM
- What I already tried: schedule 调整、loss reweighting
- What didn't work: 仅调采样温度没有本质改善

## Constraints
- Compute: 200 GPU-hours
- Timeline: 4 months
- Target venue: NeurIPS 2026

## What I'm Looking For
- [x] Improvement on existing method
- [ ] New direction from scratch

## Non-Goals
不做超大规模预训练
```

### 11.5 启动 Codex 会话

```bash
codex -C /data_3/wly/research/ddlm-gap --search -s workspace-write -a on-request
```

进入后，建议先输入一段预热指令：

```text
请先阅读当前项目的 AGENTS.md 和 RESEARCH_BRIEF.md，确认可用的 ARIS Codex skills，并告诉我你将按 ARIS 的文件流来工作。
```

#### 预期行为

Codex 会：

- 先读这两个文件
- 识别当前项目约束
- 理解这是一个 ARIS 风格项目
- 准备后续调用 skill

### 11.6 第一步正式操作：找 idea

你输入：

```text
/idea-discovery "离散扩散语言模型中的 factorized gap" — AUTO_PROCEED: false, arxiv download: true
```

#### 预期内部行为

Codex 会先：

- 调 `research-lit`
- 再调 `idea-creator`
- 然后调 `novelty-check`
- 再调 `research-review`
- 最后调 `research-refine-pipeline`

其中 `research-review` 这类 review-heavy 环节会用次级 Codex agent 做 reviewer。

#### 预期输出文件

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`

#### 你此时应该做什么

阅读 `IDEA_REPORT.md` 和 `FINAL_PROPOSAL.md`，确认当前方案是不是你接受的。

### 11.7 第二步：固化当前方案

你输入：

```text
请根据 IDEA_REPORT.md 和 refine-logs/FINAL_PROPOSAL.md 创建 docs/research_contract.md，只保留当前最优方案。
```

#### 预期行为

Codex 会生成一页聚焦文档，减少后面 session 污染。

#### 预期输出

- `docs/research_contract.md`

### 11.8 第三步：桥接到实验

你输入：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md"
```

#### 预期行为

Codex 会：

- 解析实验计划
- 检查现有代码
- 补齐缺的训练 / 评测脚本
- 先做 sanity-stage
- 再调用 `/run-experiment`

#### 预期写入

- 代码实现文件
- `EXPERIMENT_LOG.md`
- `refine-logs/EXPERIMENT_TRACKER.md`
- `results/`

### 11.9 第四步：查看实验状态

你输入：

```text
/monitor-experiment
```

#### 预期行为

Codex 会：

- 检查远程 `screen`
- 检查日志
- 检查结果文件
- 更新 tracker

#### 预期输出

- 更新后的 `refine-logs/EXPERIMENT_TRACKER.md`
- 追加后的 `EXPERIMENT_LOG.md`

### 11.10 第五步：把结果翻译成 claim

你输入：

```text
/result-to-claim
```

#### 预期行为

Codex 会分析：

- 哪些结果支持你的主 claim
- 哪些只能支撑弱 claim
- 哪些结果说明方法需要收缩表述或继续补实验

#### 预期产物

- 对 `findings.md` 的更新
- 对 claim 状态的记录
- 可选的 `CLAIMS_FROM_RESULTS.md`

### 11.11 第六步：进入自动 review 循环

你输入：

```text
/auto-review-loop "离散扩散语言模型中的 factorized gap" — human checkpoint: true
```

#### 预期内部行为

Round 1：

1. 主 agent 汇总当前研究上下文。
2. 用 `spawn_agent` 创建 reviewer agent。
3. reviewer 给出分数、弱点和最小修复集合。
4. 主 agent 将 reviewer 的原始回复写入 `AUTO_REVIEW.md`。
5. 主 agent 根据建议改代码、补实验、改表述。
6. 记录 `REVIEW_STATE.json`。

Round 2+：

1. 主 agent 用 `send_input` 把增量信息发给同一个 reviewer 线程。
2. reviewer 根据新结果继续评审。
3. 主 agent 重复修复。

#### 预期核心文件

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`

### 11.12 第七步：生成写作输入

你输入：

```text
请根据 docs/research_contract.md、EXPERIMENT_LOG.md、AUTO_REVIEW.md 和最新结果，生成 NARRATIVE_REPORT.md。
```

#### 预期输出

- `NARRATIVE_REPORT.md`

### 11.13 第八步：写论文

你输入：

```text
/paper-writing "NARRATIVE_REPORT.md" — venue: NeurIPS
```

#### 预期行为

Codex 会依次跑：

- `paper-plan`
- `paper-figure`
- `paper-write`
- `paper-compile`
- `auto-paper-improvement-loop`

#### 预期输出

- `PAPER_PLAN.md`
- `paper/main.tex`
- `paper/main.pdf`
- `paper/PAPER_IMPROVEMENT_LOG.md`

### 11.14 第九步：可选生成报告材料

你输入：

```text
/paper-slides "paper/"
/paper-poster "paper/"
```

#### 预期输出

- 幻灯片文稿
- 海报文件

### 11.15 这一整套模拟之后，项目目录会长什么样

```text
ddlm-gap/
├── AGENTS.md
├── RESEARCH_BRIEF.md
├── IDEA_REPORT.md
├── EXPERIMENT_LOG.md
├── AUTO_REVIEW.md
├── REVIEW_STATE.json
├── NARRATIVE_REPORT.md
├── PAPER_PLAN.md
├── docs/
│   └── research_contract.md
├── refine-logs/
│   ├── FINAL_PROPOSAL.md
│   ├── EXPERIMENT_PLAN.md
│   └── EXPERIMENT_TRACKER.md
├── results/
├── papers/
└── paper/
    ├── main.tex
    ├── main.pdf
    ├── sections/
    ├── figures/
    └── PAPER_IMPROVEMENT_LOG.md
```

## 12. 如果你是第一次用 ARIS，而且现在只有 Codex，最推荐你怎么跑通第一次全流程

这一节回答你的第三个核心要求：

**你作为一个刚接触 ARIS 的小白，且现在只有 Codex，我建议你怎样第一次跑通全流程、同时学会 ARIS？**

答案很明确：

- 不要一开始就追求“最高自动化”
- 先跑一条“可理解、可检查、能看到文件流”的学习路线
- 等你理解了文件和 skill 的关系，再上 `/research-pipeline`

### 12.1 你第一次体验的目标应该是什么

第一次体验的目标不是：

- 一次性做出真正可投稿成果

而应该是：

1. 学会如何启动纯 Codex ARIS 项目
2. 学会如何写 `AGENTS.md`
3. 学会如何调用 `idea-discovery`
4. 学会看懂 `IDEA_REPORT.md`、`EXPERIMENT_PLAN.md`、`AUTO_REVIEW.md`
5. 学会理解“一个阶段的输出，如何成为下一阶段的输入”

只要你把这几点走通了，你就真正学会 ARIS 了。

### 12.2 你第一次体验时，不建议用太大的题目

不要第一次就上：

- 大模型预训练
- 需要上千 GPU-hours 的课题
- 需要新数据清洗的大任务

第一次更适合：

- 一个相对明确的改进型问题
- 有公开 baseline
- 有现成代码基础
- 可在中小预算下做 sanity 验证

这也是为什么我在文档里一直用“离散扩散语言模型的 factorized gap”这种结构化问题作示例。

### 12.3 你第一次最推荐的命令顺序

这是我最推荐你的第一次手动体验路线。

#### Shell 准备

```bash
cd /data_3/wly/ARIS/Auto-claude-code-research-in-sleep
mkdir -p ~/.codex/skills
cp -a skills/skills-codex/* ~/.codex/skills/

mkdir -p /data_3/wly/research/aris-first-run
cd /data_3/wly/research/aris-first-run
mkdir -p docs refine-logs results paper papers
```

#### 写配置文件

在项目根目录中写：

- `AGENTS.md`
- `RESEARCH_BRIEF.md`
- 可选 `codex.md`

#### 启动 Codex

```bash
codex -C /data_3/wly/research/aris-first-run --search -s workspace-write -a on-request
```

#### 会话中的推荐顺序

1. `请先阅读 AGENTS.md 和 RESEARCH_BRIEF.md，确认当前项目状态与可用 skills。`
2. `/idea-discovery "你的研究主题" — AUTO_PROCEED: false`
3. `请根据输出创建 docs/research_contract.md`
4. `/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md"`
5. `/monitor-experiment`
6. `/result-to-claim`
7. `/auto-review-loop "你的研究主题" — human checkpoint: true`
8. `请生成 NARRATIVE_REPORT.md`
9. `/paper-writing "NARRATIVE_REPORT.md" — venue: NeurIPS`

### 12.4 为什么推荐这条顺序

#### 第 1 步先读配置

原因：

- 避免 Codex 直接在错误假设下开工
- 让它先知道服务器、预算和当前阶段

#### 第 2 步先跑 `idea-discovery`

原因：

- 这是 ARIS 最典型的入口
- 它能最明显地展示“多 skill 编排 + 文件化输出”的特征

#### 第 3 步补 `research_contract`

原因：

- 让项目上下文从“很多候选 idea”收敛成“一个当前活跃方案”

#### 第 4 到 6 步进入实验和 claim 阶段

原因：

- 让你看到计划如何变成代码与结果
- 让你理解 ARIS 为什么不是只会写 prompt

#### 第 7 步再进入 auto review

原因：

- 这是 ARIS 体现“执行者 / reviewer 分离”的关键阶段
- 此时你已经有了足够上下文，不会空跑

#### 第 8 到 9 步最后写论文

原因：

- 论文写作应该建立在已经收敛的故事和结果上

### 12.5 第一次体验时，每一步你应该重点观察什么

#### `idea-discovery` 后

重点看：

- `IDEA_REPORT.md` 是否有清晰排名
- `FINAL_PROPOSAL.md` 是否明确了方法
- `EXPERIMENT_PLAN.md` 是否围绕 claim 展开

#### `experiment-bridge` 后

重点看：

- 是否真的生成了实验代码
- `EXPERIMENT_TRACKER.md` 是否在更新
- `EXPERIMENT_LOG.md` 是否记录了实验摘要

#### `auto-review-loop` 后

重点看：

- `AUTO_REVIEW.md` 是否完整记录每一轮 reviewer 意见
- `REVIEW_STATE.json` 是否可作为断点恢复依据
- reviewer 是不是在真正推动改进，而不是只给表扬

#### `paper-writing` 后

重点看：

- `PAPER_PLAN.md` 是否有 story structure
- `paper/main.tex` 是否形成完整论文骨架
- `paper/main.pdf` 是否成功编译

### 12.6 第一次体验时，哪些参数建议打开

我建议你第一次尽量保守：

```text
AUTO_PROCEED: false
human checkpoint: true
compact: true
```

原因分别是：

- `AUTO_PROCEED: false`
  让你在 idea 选择关卡停下来，不会自动一路冲下去
- `human checkpoint: true`
  让你在 review 每轮后都能观察系统行为
- `compact: true`
  让输出文件更利于恢复和阅读

### 12.7 第一次体验时，不建议你做的事

第一次先不要：

- 一开始就用 `/research-pipeline`
- 一开始就让系统跑很多 GPU-hours 的大实验
- 一开始就叠加 Zotero、Obsidian、W&B、Gemini overlay 等大量外设
- 一开始就追求一夜全自动投顶会

先把基础闭环走通，比什么都重要。

## 13. 当你第一次分步流程跑通后，下一次应该怎么体验“真正的全自动”

当你已经完成一次分步体验，理解了：

- `AGENTS.md` 怎么写
- 各阶段会生成哪些文件
- `auto-review-loop` 怎么记录状态
- `paper-writing` 会产出什么

这时你就可以尝试更自动化的路径。

### 13.1 半自动版

```bash
codex -C /你的项目目录 --search -s workspace-write -a on-request
```

进入后输入：

```text
/idea-discovery "你的研究方向" — AUTO_PROCEED: false, compact: true
```

等你确认 top idea 后，再继续：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
/auto-review-loop "你的研究方向" — human checkpoint: true, compact: true
/paper-writing "NARRATIVE_REPORT.md"
```

### 13.2 更自动的总控版

当你已经完全理解系统以后，可以尝试：

```text
/research-pipeline "你的研究方向" — AUTO_PROCEED: false, human checkpoint: true
```

这条命令会更像“总导演”，把 workflow 1、实验实现和 workflow 2 串起来。

但我仍然建议：

- 第一次自动总控时也保留 checkpoint
- 不要一上来就完全无人值守

## 14. 纯 Codex 路径的几个关键提醒

### 14.1 `AGENTS.md` 对纯 Codex 比 `CLAUDE.md` 更重要

因为很多 Codex 版 skill 明确是去读 `AGENTS.md`。

### 14.2 `research-wiki` 不是纯 Codex 基础包的默认组成部分

所以第一次别把它当成必经步骤。

### 14.3 `code_sync: git` 的前提是你的研究项目本身必须是 Git 仓库

否则部署会不稳定。

### 14.4 没有 `--search`，很多文献相关技能会明显变弱

所以纯 Codex 路径启动时，建议带：

```text
--search
```

### 14.5 第一次体验时，最好保留人工 checkpoint

这样你能真正看懂系统，而不是只看到“它跑完了”。

## 15. 结语

如果把这份纯 Codex 手册压缩成一句话，那就是：

**纯 Codex 版 ARIS 并不是“把 Claude 去掉就没法用”，而是把 Codex 本身同时用作执行者和次级 reviewer，通过 `skills-codex`、`AGENTS.md` 和项目状态文件，把完整科研流程跑起来。**

对你当前这个阶段，最重要的行动顺序只有 6 步：

1. 把 `skills/skills-codex/*` 安装到 `~/.codex/skills/`
2. 新建独立研究项目目录
3. 写 `AGENTS.md`
4. 写 `RESEARCH_BRIEF.md`
5. 用 `codex -C <project> --search -s workspace-write -a on-request` 启动
6. 先按分步流程体验一次，再考虑总控自动化

如果你愿意，我下一步可以不只停留在写手册，而是直接继续帮你做两件事中的任意一件：

1. 在某个新的研究项目目录里，替你把 `AGENTS.md`、`RESEARCH_BRIEF.md`、`codex.md` 初始化好，形成一套可直接开跑的纯 Codex ARIS 模板。
2. 直接带你在一个具体课题上，按这份手册一步一步开始第一次真实的纯 Codex 自动科研体验。

## 16. 补充答疑：针对第一次阅读本手册时最容易困惑的 7 个问题

这一节专门回答你刚才提出的 7 个问题。

### 16.1 问题 1：我在项目目录里启动 Codex 后，它真的知道“ARIS 的文件流”是什么吗？

短答案：

- **不是天然知道**
- **也不是因为它在项目目录里就自动知道**
- 它之所以能理解 ARIS 文件流，是因为你已经把 ARIS 的 Codex 技能安装到了 `~/.codex/skills/`，而这些 skill 的 `SKILL.md` 里写明了输入输出契约

也就是说，“ARIS 的文件流”并不是一个神秘内置能力，而是由三部分组成的：

1. `~/.codex/skills/` 里的 `SKILL.md`
2. 你的项目文件约定
3. 你在启动会话时给它的引导

例如：

- `idea-discovery` 明确会产出 `IDEA_REPORT.md`、`refine-logs/FINAL_PROPOSAL.md`、`refine-logs/EXPERIMENT_PLAN.md`
- `experiment-bridge` 明确会读取 `EXPERIMENT_PLAN.md`、`FINAL_PROPOSAL.md`
- `auto-review-loop` 明确会写 `AUTO_REVIEW.md`、`REVIEW_STATE.json`
- `paper-writing` 明确会写 `paper/`

所以更准确的说法不是“Codex 一启动就懂 ARIS 文件流”，而是：

**Codex 在安装了 ARIS Codex 技能后，可以通过读取这些 skill 的工作流说明，逐步建立对 ARIS 文件流的理解。**

因此，我建议把原来那句“预热指令”理解成“让它先建立 ARIS 语境”，而不是“它已经全知道了”。

更稳妥的预热指令可以改成下面这样：

```text
请先阅读当前项目的 AGENTS.md 和 RESEARCH_BRIEF.md，再检查已安装的 ARIS Codex skills，重点确认 /idea-discovery、/experiment-bridge、/auto-review-loop、/paper-writing 这四个 skill 的输入输出文件契约，并按这些文件契约在当前项目中工作。
```

这句比“按 ARIS 的文件流来工作”更明确，因为它直接告诉 Codex：

- 去看哪些 skill
- 去看哪些文件契约
- 在当前项目中执行

### 16.2 问题 2：为什么它会自动识别当前工作目录里的 `AGENTS.md` 和 `RESEARCH_BRIEF.md`？如何保证读写都在当前路径下？

这里要分成两部分看。

#### 第一部分：为什么它会优先在当前项目目录里工作

因为你是这样启动的：

```bash
codex -C /你的项目目录 --search -s workspace-write -a on-request
```

这里的 `-C /你的项目目录` 的作用，是把这个目录设成 Codex 当前会话的工作根目录。

这意味着：

- 相对路径默认都从这个目录开始解释
- skill 里提到的 `AGENTS.md`、`results/`、`paper/`、`refine-logs/`，默认都指向当前项目根目录下的这些路径
- shell 命令默认也在这个目录执行，除非它显式切换目录

#### 第二部分：为什么它会读 `AGENTS.md` 和 `RESEARCH_BRIEF.md`

不是所有 skill 都会“自动显式读取这两个文件”，而是：

- 有些 skill 明确写了要读 `AGENTS.md`
  例如 `run-experiment`、`research-lit`
- 有些高层 skill 没有逐字写死文件名，但它们会基于当前项目上下文工作
- 如果你在开始时用预热指令让它先读 `AGENTS.md` 和 `RESEARCH_BRIEF.md`，后续高层 skill 的行为会更稳定

所以，正确理解应该是：

- `AGENTS.md`：很多 Codex 版 skill 的**显式配置输入**
- `RESEARCH_BRIEF.md`：高层工作流的**研究背景输入**

#### 第三部分：如何进一步保证所有读写都落在当前项目目录

严格说，`-C` 是默认工作根目录，但**不是绝对强制防呆结界**。如果你主动在 prompt 中要求去别的绝对路径写文件，Codex 当然也可能照做。

为了最大程度保证路径稳定，我建议你第一次会话开始时加一句：

```text
后续所有 ARIS 输出默认写在当前项目根目录及其子目录中；不要写回 ARIS 系统仓库本身，除非我明确要求。
```

如果你想更稳一点，可以再加一句：

```text
请先执行 pwd，并确认当前工作根目录是这个研究项目目录，再开始后续步骤。
```

### 16.3 问题 3：`docs/research_contract.md` 后续 skill 会自动识别吗，还是要靠我再提示？

短答案：

- **不能假设所有 skill 都会自动优先读取它**
- 但它非常值得维护，因为它能显著提升后续 session 的稳定性

更准确的说法是：

#### 自动识别层面

有些高层 skill 会读取“项目叙事文件 / 项目记忆文件 / 当前上下文文件”，在这种情况下，`docs/research_contract.md` 很可能会被当成一个有价值的当前方案文档。

但是：

- 不是每个 skill 的 `SKILL.md` 都明确硬编码写了 `docs/research_contract.md`
- 所以你**不能把它当作百分之百自动读取的保底机制**

#### 实践层面最稳妥的做法

我建议你用三层方法确保它被持续利用：

1. 在 `AGENTS.md` 的 `Pipeline Status` 中注明当前活跃上下文文件
   例如：
   ```markdown
   ## Pipeline Status
   - Current stage: implementation
   - Active context doc: `docs/research_contract.md`
   ```

2. 在 `codex.md` 中写入恢复规则
   例如：
   ```markdown
   - 新会话优先读取 docs/research_contract.md（如果存在）
   ```

3. 在进入关键下一步时，显式再提示一次
   例如：
   ```text
   后续请以 docs/research_contract.md 作为当前活跃方案的主上下文，而不是重新把整个 IDEA_REPORT.md 全量展开。
   ```

所以结论是：

- `docs/research_contract.md` 非常重要
- 但为了稳定，**第一次几轮关键技能调用时，最好在 prompt 中再点名一次**

### 16.4 问题 4：实验运行时，我还能输入 `/monitor-experiment` 吗？会不会中断正在跑的东西？

这个问题非常关键。

短答案：

- **如果实验已经在远程 `screen` / 后台 detached 运行，Codex 会话被打断通常不会中断实验本身**
- **但如果当前 assistant 还在同一个回合里忙于执行 `experiment-bridge`，你直接插话可能会中断这次对话推理流程**

你要把“Codex 会话”和“远程实验进程”分开看。

#### 情况 A：实验已经通过 `screen` 成功提交到远程服务器

这时远程实验是独立进程。

即使：

- 你中断当前 Codex 会话
- 终端暂时断联
- 你稍后重新进来

通常远程 `screen` 里的实验仍然继续跑。

这也是为什么 ARIS 的 `run-experiment` / `experiment-bridge` 设计里强调：

- 用 `screen`
- 用后台 detached 方式
- 用 `tee` 落日志

#### 情况 B：`experiment-bridge` 还在同一轮里做代码修改、汇总、部署前检查

这时你如果突然插入新指令，可能会：

- 中断当前这一次 assistant 的推理和写文件动作
- 让它来不及把本轮总结写完

但是否中断**远程实验**，取决于远程实验是否已经真正被启动。

#### 最推荐的操作方式

第一次体验时，不建议你在 `experiment-bridge` 正进行到一半时强行插入 `/monitor-experiment`。

更稳妥的顺序是：

1. 让 `/experiment-bridge` 完成“实现 + 部署 + 返回摘要”
2. 然后再输入：
   ```text
   /monitor-experiment
   ```

#### 如果你确实想在实验运行期间持续查看状态

有两个更稳妥的方案。

方案 1：

- 等 `experiment-bridge` 这一轮响应结束后
- 在同一会话继续输入 `/monitor-experiment`

方案 2：

- 另开一个新终端
- 重新进入同一项目目录
- 启动一个新的 Codex 会话或恢复会话
- 专门用于监控

例如：

```bash
codex resume --last -C /你的项目目录 --search -s workspace-write -a on-request
```

或者新开一轮：

```bash
codex -C /你的项目目录 --search -s workspace-write -a on-request
```

然后告诉它：

```text
请只做实验监控，不改写当前方案。先读取 AGENTS.md、refine-logs/EXPERIMENT_TRACKER.md 和已有日志，然后执行 /monitor-experiment。
```

### 16.5 问题 5：Codex 只有 1M token，上下文会不会后面全丢掉？

短答案：

- **会发生上下文压缩**
- **但 ARIS 本来就不是靠聊天上下文长期保存项目状态**

ARIS 的设计哲学本来就是：

- 重要状态落文件
- 会话只是当前执行层

所以，只要你把阶段性结果持续写回文件，1M token 并不是致命问题。

#### 为什么仍然可能出问题

如果你做了以下事情：

- 一直只靠聊天，不写文件
- 不维护 `docs/research_contract.md`
- 不维护 `AUTO_REVIEW.md`
- 不维护 `EXPERIMENT_LOG.md`
- 不更新 `Pipeline Status`

那么一旦上下文压缩，多轮历史很可能就会丢失细节。

#### 为什么 ARIS 还能继续跑

因为它的关键状态本来就应该写入这些文件：

- `AGENTS.md` 中的 `Pipeline Status`
- `docs/research_contract.md`
- `IDEA_CANDIDATES.md` 或 `IDEA_REPORT.md`
- `EXPERIMENT_LOG.md`
- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- `NARRATIVE_REPORT.md`

#### 你应该怎么做来降低上下文丢失风险

我建议你第一次体验就坚持下面这些做法：

1. 打开 `compact: true`
2. idea 选定后立刻生成 `docs/research_contract.md`
3. 每轮 review 后确保 `AUTO_REVIEW.md` 和 `REVIEW_STATE.json` 更新
4. 每完成实验就更新 `EXPERIMENT_LOG.md`
5. 在 `AGENTS.md` 里维护 `Pipeline Status`
6. 新 session 进来时先读这些文件，而不是依赖聊天历史

换句话说：

**1M token 是会话缓存，不是项目数据库。ARIS 的项目数据库就是这些 Markdown / JSON 状态文件。**

### 16.6 问题 6：如果 Codex 服务不稳定、突然断联，会打断 ARIS 吗？能从断点恢复吗？

短答案：

- **会打断“当前会话中的推理”**
- **不一定会打断“已经发出去的远程实验”**
- **可以恢复，但前提是你把状态写进了文件**

#### 哪些东西会被打断

可能会被打断的是：

- 当前这一轮 assistant 还没来得及写完的推理
- 当前这一步还没落盘的中间结论
- 正在前台运行、没有 detached 的长命令

#### 哪些东西通常不会被打断

通常不会被打断的是：

- 已经在远程 `screen` 中启动的训练
- 已经写入磁盘的 `IDEA_REPORT.md`
- 已经写入磁盘的 `AUTO_REVIEW.md`
- 已经写入磁盘的 `REVIEW_STATE.json`
- 已经写入磁盘的 `EXPERIMENT_TRACKER.md`

#### 你可以怎么恢复

你这台机器上的 Codex CLI 已支持：

```bash
codex resume --last
```

所以第一种恢复方式是：

```bash
codex resume --last -C /你的项目目录 --search -s workspace-write -a on-request
```

第二种是直接开新会话，然后明确要求它按状态文件恢复：

```text
请先阅读 AGENTS.md、docs/research_contract.md、refine-logs/EXPERIMENT_TRACKER.md、EXPERIMENT_LOG.md、AUTO_REVIEW.md 和 REVIEW_STATE.json（如果存在），判断当前停在哪个阶段，然后只继续下一步，不要重复前面已经完成的步骤。
```

#### 哪些文件最关键

如果你担心断联，最关键的恢复文件是：

- `AGENTS.md`
- `docs/research_contract.md`
- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`

其中：

- `EXPERIMENT_TRACKER.md` 负责告诉你实验跑到哪
- `REVIEW_STATE.json` 负责告诉你 review loop 跑到哪

### 16.7 问题 7：`AGENTS.md`、`RESEARCH_BRIEF.md`、`codex.md` 到底分别是什么？skills 怎么调用它们？

这是纯 Codex 路径里最需要讲清楚的一个问题。

#### `AGENTS.md`：项目运行说明书

它的核心作用是：

- 告诉 Codex 这个项目的运行环境是什么
- 告诉 Codex 远程服务器怎么连
- 告诉 Codex 代码该同步到哪里
- 告诉 Codex 当前项目处在哪个阶段

它最像：

- 项目运行合同
- 项目执行操作手册

在纯 Codex ARIS 里，它是**最接近“系统配置文件”**的东西。

哪些 skill 会直接或间接用它？

- `run-experiment`：显式读取它，找 SSH、Conda、Code dir、`code_sync`、W&B
- `research-lit`：显式读取它，找本地 paper library 配置
- `experiment-bridge`：虽然其自身重点读实验计划，但它会调用 `run-experiment`，所以最终也依赖 `AGENTS.md`
- 其他高层 skill：通过当前项目上下文和后续子 skill 间接依赖它

#### `RESEARCH_BRIEF.md`：研究问题输入文档

它的核心作用是：

- 把一句太短的题目，扩展成系统能真正理解的研究任务说明

它最像：

- 研究需求书
- 课题输入说明

里面通常写：

- 问题定义
- 背景
- 已做尝试
- 失败经验
- 预算
- 时间线
- 目标 venue
- 非目标

哪些 skill 会用它？

- `idea-discovery`
- `research-pipeline`
- 任何需要理解研究背景的高层规划 skill

严格说，并不是每个 `SKILL.md` 都逐字写死“必须读取 `RESEARCH_BRIEF.md`”，但在 ARIS 的主工作流语义里，它就是标准的上游研究输入文件。

所以实践上，你应该在启动时就让 Codex 先读它。

#### `codex.md`：Codex 会话行为说明 / 恢复入口

它不是 ARIS 主线仓库规定的硬性项目产物，而是对 Codex CLI 很有用的一个辅助文件。

它最像：

- 会话恢复规则
- 局部 system prompt
- 新 session 的入口说明

它通常写：

- 新会话先读哪些文件
- 如何判断当前阶段
- 哪些文档优先级最高
- 不要重复已经完成的步骤

哪些 skill 会直接调用它？

- **通常没有 skill 会显式“调用” `codex.md`**

更准确地说：

- `AGENTS.md` 和 `RESEARCH_BRIEF.md` 更像是项目工作内容输入
- `codex.md` 更像是“宿主 Codex 如何进入这个项目”的说明

所以：

- `AGENTS.md` 是执行配置
- `RESEARCH_BRIEF.md` 是研究输入
- `codex.md` 是会话行为与恢复规则

#### 三者怎么配合最稳

最推荐的分工是：

- `AGENTS.md`：写环境、规则、当前阶段
- `RESEARCH_BRIEF.md`：写问题背景与研究约束
- `docs/research_contract.md`：写当前活跃方案
- `codex.md`：写新会话恢复顺序

这样无论：

- 会话压缩
- 网络断联
- 模型重启
- 你第二天重新进入项目

Codex 都更容易恢复到正确状态。
