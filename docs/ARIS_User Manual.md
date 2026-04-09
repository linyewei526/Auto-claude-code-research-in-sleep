# ARIS 系统详解与全流程使用手册

> 写给当前这台服务器上 `/data_3/wly/ARIS` 使用者的本地说明书。目标不是重复 README，而是把你现在这套 worktree 结构、ARIS 的真实架构、环境基础、技能调用方式、运行机制、文件流转关系和一次完整自动科研演练串成一份能直接照着做的中文手册。

## 0. 文档定位与阅读建议

本手册解决 6 个问题：

1. 你当前 `~/ARIS` 目录到底是什么结构，为什么不是一个单独仓库，而是多个 worktree。
2. ARIS 到底是什么，是“一个程序”还是“一套技能工作流”。
3. ARIS 运行时到底依赖什么：Claude Code、Codex MCP、Python 脚本、远程 GPU、LaTeX、W&B、Zotero、Obsidian、飞书等分别起什么作用。
4. 仓库每个分支、每个顶层子目录、每类文件分别承担什么职责。
5. 你实际要怎么调用 skill、怎么启动工作流、模型之间如何交互、日志与中间结果存到哪里。
6. 如果你要从零体验一次自动科研，全流程应该怎样开始、怎样推进、每一步会产出什么。

建议按这个顺序阅读：

- 第 1 章：先搞清你当前仓库布局。
- 第 2 章：再理解 ARIS 的总架构。
- 第 3-5 章：看代码库组织、技能体系、环境依赖。
- 第 6-9 章：看你实际怎么运行、怎么维护状态、怎么接外部工具。
- 第 10 章：照着范例走一遍完整自动科研流程。

## 1. 你当前这套 `~/ARIS` 的真实结构

首先要纠正一个非常关键的认知：**你现在的 `/data_3/wly/ARIS` 不是一个普通“单仓库根目录”**，它是一个**worktree 容器目录**。真正的 Git 主仓库是：

- `/data_3/wly/ARIS/Auto-claude-code-research-in-sleep`

而同级的其他目录是同一个 Git 仓库的**额外工作树**。你当前机器上实际是这样：

```text
/data_3/wly/ARIS
├── Auto-claude-code-research-in-sleep   # main 分支工作树
├── aris-code                            # aris-code 分支工作树
├── paper-poster                         # feature/paper-poster 分支工作树
├── paper-slides                         # feature/paper-slides 分支工作树
└── personal                             # personal 分支工作树
```

对应的 `git worktree list` 结果是：

```text
/data_3/wly/ARIS/Auto-claude-code-research-in-sleep  62e3777 [main]
/data_3/wly/ARIS/aris-code                           4c498b3 [aris-code]
/data_3/wly/ARIS/paper-poster                        a430df6 [feature/paper-poster]
/data_3/wly/ARIS/paper-slides                        d23e3cd [feature/paper-slides]
/data_3/wly/ARIS/personal                            0c2e083 [personal]
```

这意味着：

- `~/ARIS` 根目录本身不是 Git 仓库，所以你在这里直接 `git branch` 会报错。
- 你必须进入某个 worktree 目录，例如 `~/ARIS/Auto-claude-code-research-in-sleep`，再执行 Git 命令。
- 这 5 个目录共享同一个 Git 历史，但每个目录可以固定在不同分支上，便于同时查看多个分支的内容。

### 1.1 这 5 个 worktree 各自是什么

#### 1. `Auto-claude-code-research-in-sleep`

这是主线工作树，也就是你现在最应该理解和使用的目录。它承载：

- 主 README 与绝大多数中文/英文文档
- 主技能系统 `skills/`
- MCP 桥接脚本 `mcp-servers/`
- 模板 `templates/`
- 辅助脚本 `tools/`
- 测试 `tests/`

你平时如果要：

- 安装 skill 到 Claude Code
- 阅读 ARIS 主文档
- 修改主线技能
- 跑主线工作流

都应该主要在这个目录下进行。

#### 2. `aris-code`

这是独立 CLI 分支。它不是简单的 `SKILL.md` 集合，而是一个 **Rust 实现的 ARIS-Code 命令行程序**。它解决的是：

- 不再完全依赖 Claude Code 官方 CLI
- 将 skills、session、配置、reviewer、日志、memory、权限模式等做成独立 REPL
- 内置 `LlmReview`、`Skill`、`TodoWrite` 等工具

简言之：

- `Auto-claude-code-research-in-sleep` 更像“工作流与技能仓库”
- `aris-code` 更像“把这些能力产品化为独立 CLI 的实现仓库”

#### 3. `paper-slides`

这是 `feature/paper-slides` 分支的工作树，历史上用于引入 `paper-slides` 技能及相关文档。你当前快照里，主线 `main` 已经包含了 `paper-slides` 能力，因此这个 worktree 的价值主要变成：

- 保留该功能的独立分支历史
- 便于单独查看分支语义
- 如果你要研究这个功能是如何合并进主线的，可以在这里看

#### 4. `paper-poster`

与上面同理，这是 `feature/paper-poster` 分支的工作树。当前主线也已经包含了 `paper-poster` 技能，因此它现在更像：

- 历史分支工作树
- 功能演化快照
- 独立查看 poster 相关修改的地方

#### 5. `personal`

这是你自己的个人分支工作树，用来保存：

- 个人文档
- 配置备忘
- 你自己的理解总结
- 未来你自己对 ARIS 的定制说明

你要求我写的文档就应该落在这里，因为：

- 不污染主线
- 方便你长期保留自己的使用手册
- 后续即使拉取上游更新，也不影响你的个人文档

### 1.2 你应该如何理解“分支”和“目录”的关系

在你这套布局里，**目录名基本就是分支角色的具象化**：

- 目录不是“模块分拆”
- 目录是“同一仓库不同分支的并排展开”

所以你的分析视角应该是：

- 第一层：先区分这是哪个分支的 worktree
- 第二层：再看这个分支内部有哪些子目录

如果你把 `~/ARIS` 误当成“一个大项目的五个子模块”，后面会一直混乱。

### 1.3 你接下来主要应该用哪一个目录

对你当前“想真正把 ARIS 用起来”的目标，优先级应当是：

1. **主入口**：`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep`
2. **参考独立 CLI 实现**：`/data_3/wly/ARIS/aris-code`
3. **个人记录与定制**：`/data_3/wly/ARIS/personal`
4. **历史功能分支参考**：`paper-slides`、`paper-poster`

如果只想尽快跑一次完整流程，实际上你只需要：

- 主线目录
- 你的个人目录

其余两个功能分支现在不是运行必需品。

## 2. ARIS 到底是什么

一句话定义：

> **ARIS 不是一个“单体自动科研程序”，而是一套以 Markdown skill 为核心、以文件状态为中枢、以执行模型 + 审稿模型对抗协作为方法论的自动科研工作流系统。**

这句话要拆开理解。

### 2.1 ARIS 不是“一个 Python 框架”

ARIS 最核心的设计选择是：**把工作流写成技能说明，而不是写成重框架**。

也就是说，主线仓库最重要的不是某个 `.py` 主程序，而是这些内容：

- `skills/*/SKILL.md`
- `templates/*.md`
- `docs/*.md`
- `tools/*.py`
- `mcp-servers/*`

其运行哲学是：

1. 用 `SKILL.md` 描述工作流步骤、约束、检查点、输入输出文件约定。
2. 让执行 Agent（通常是 Claude Code）去读这些技能说明。
3. 让执行 Agent 按说明调用工具、改代码、读写文件、跑实验。
4. 在关键环节引入另一个模型做“外部审稿人”，避免单模型自审。

所以你必须明白：**ARIS 的核心资产是“方法论 + 技能提示词 + 文件协议”**，不是传统意义上的后端服务。

### 2.2 ARIS 也不是“只靠一句 prompt”

仓库 README 开头会给人一种“直接 `/research-pipeline "某方向"` 就完事”的感觉，但实际不是“神奇一句话自动完成科研”，而是：

- 一个总 skill 会继续调用多个子 skill
- 各子 skill 会不断落盘中间文件
- 文件成为后续步骤的输入
- 审稿模型不断挑错
- 执行模型不断修正

所以 ARIS 的真实运行单元是：

```text
用户命令
→ Skill 指令
→ 工具调用
→ 文件产物
→ 下一个 Skill 读取这些文件
→ 继续推进
```

### 2.3 ARIS 的核心思想：执行者与审稿人分离

这是 ARIS 最重要的架构思想。

系统角色分成两类：

#### 执行者 Executor

负责：

- 读代码
- 改代码
- 写实验脚本
- 部署训练
- 收结果
- 写论文
- 维护项目状态文件

主线默认设想的执行者是：

- Claude Code

也可以替换为：

- Codex CLI
- Cursor
- Trae
- Antigravity
- OpenClaw
- 其他能读 `SKILL.md` 并调用工具的 Agent

#### 审稿人 Reviewer

负责：

- 对 idea、方法、实验、论文、rebuttal 做外部批判
- 给分
- 找弱点
- 指定最小修复项
- 在某些模式下保留跨轮次“怀疑记忆”

默认 reviewer 常见组合有：

- Codex MCP / GPT-5.4
- Claude-review bridge
- Gemini-review bridge
- llm-chat MCP 对接 GLM / DeepSeek / MiniMax / Kimi 等

ARIS 之所以强调“双模型”，是因为它不希望：

- 同一个模型一边写，一边自己夸自己
- 最终陷入“看起来合理，实际上漏洞很多”的局部最优

### 2.4 ARIS 的核心不是聊天记录，而是文件状态

这点对真正使用 ARIS 极重要。

ARIS 并不把“对话历史”当作唯一记忆，而是主动把关键状态写进项目文件。例如：

- `CLAUDE.md` 里的 `Pipeline Status`
- `IDEA_REPORT.md`
- `IDEA_CANDIDATES.md`
- `docs/research_contract.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `EXPERIMENT_LOG.md`
- `findings.md`
- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- `research-wiki/`

这意味着：

- 你换会话后还能恢复工作
- 上下文被压缩后不至于完全失忆
- 长流程能跨夜运行
- 论文/实验/idea 的状态不依赖单次聊天窗口

所以你要把 ARIS 看成：

> 一个“由 Agent 执行、由 Markdown/JSON 文件持久化状态、由多 skill 串联”的科研操作系统雏形。

## 3. ARIS 的总架构

把 ARIS 抽象成系统图，可以分为 6 层。

```text
第 1 层：用户入口层
  你在 Claude Code / Codex CLI / ARIS-Code CLI / Cursor / Trae 中下指令

第 2 层：技能编排层
  skills/*/SKILL.md 定义工作流、子步骤、检查点、输入输出契约

第 3 层：执行与审稿层
  Executor 负责做事
  Reviewer 负责独立打分、挑错、施压

第 4 层：工具与桥接层
  Bash / Read / Write / Edit / Grep / WebSearch / WebFetch
  mcp__codex__codex
  llm-chat / gemini-review / claude-review / zotero / obsidian-vault

第 5 层：项目状态与产物层
  IDEA_REPORT.md / EXPERIMENT_PLAN.md / AUTO_REVIEW.md / paper/ / research-wiki/

第 6 层：辅助脚本与基础设施层
  tools/*.py / mcp-servers/* / 远程 GPU / screen / tmux / W&B / LaTeX / Feishu
```

### 3.1 从一次 `/research-pipeline` 命令看架构

如果你在 Claude Code 中输入：

```text
/research-pipeline "离散扩散语言模型的 factorized gap"
```

大致会发生这些事情：

1. Claude Code 识别到 `/research-pipeline` 是已安装 skill。
2. Claude Code 读取对应 `SKILL.md`。
3. `research-pipeline` 再要求执行者去调用：
   - `/idea-discovery`
   - 实现实验
   - `/run-experiment`
   - `/auto-review-loop`
4. 在 `idea-discovery` 内部又会继续调用：
   - `/research-lit`
   - `/idea-creator`
   - `/novelty-check`
   - `/research-review`
   - `/research-refine-pipeline`
5. 每个阶段把结果写到项目文件。
6. 之后的技能读取前面生成的文件继续推进。
7. 评审环节通过 Codex MCP 或其他 reviewer bridge 获取外部模型反馈。
8. 如果需要实验，执行者会根据 `CLAUDE.md` 的 GPU/SSH 配置去远程部署。
9. 结果回来后继续审稿、修正、写论文。

也就是说，**一个顶层命令背后其实是一条技能调用链 + 文件状态流转链**。

### 3.2 ARIS 的四种核心构件

#### A. Skill

Skill 是 ARIS 的编排单元，通常一个目录一个技能：

```text
skills/idea-discovery/
└── SKILL.md
```

`SKILL.md` 里面一般会定义：

- skill 名称与描述
- 适用场景
- 可用工具
- 参数默认值
- 工作流阶段
- 输入文件
- 输出文件
- 错误处理原则
- 与其他 skill 的组合方式

#### B. MCP / 外部工具

Skill 自己不会“凭空联网”或“凭空调第二模型”，它要通过：

- Claude Code 自带工具
- MCP server
- 本地 Python 辅助脚本

来实现能力扩展。

例如：

- `mcp__codex__codex`：调用 Codex 作为审稿人
- `mcp__zotero__*`：搜索 Zotero
- `mcp__obsidian-vault__*`：搜索 Obsidian 笔记
- `llm-chat`：把任意 OpenAI-compatible API 作为 reviewer
- `claude-review` / `gemini-review`：本地桥接 reviewer

#### C. 项目状态文件

ARIS 不依赖隐式记忆，而依赖明确文件。

这类文件承担两种职责：

- 让下游 skill 有标准输入
- 让新会话能恢复状态

#### D. 辅助脚本

有些任务不适合完全交给 LLM 自己硬写，因此仓库提供了稳定脚本：

- `tools/arxiv_fetch.py`
- `tools/semantic_scholar_fetch.py`
- `tools/research_wiki.py`
- `tools/watchdog.py`
- `tools/meta_opt/*.sh`

这些脚本承担的是“确定性较强”的机械工作。

### 3.3 ARIS 的两种存在形态

你必须区分下面两种形态。

#### 形态 1：Markdown 技能仓库

这就是 `Auto-claude-code-research-in-sleep` 主分支的主体。

特点：

- 轻量
- 可移植
- 对宿主 Agent 依赖较强
- 适合 Claude Code、Codex CLI、Cursor、Trae 等已有 Agent 环境

#### 形态 2：独立 CLI 产品

这就是 `aris-code` 分支。

特点：

- 自带 REPL
- 自带 session 管理
- 自带配置文件与模型切换
- 自带技能优先级解析
- 自带 reviewer 工具封装

前者更像“技能仓库”，后者更像“可执行产品”。

### 3.4 你应该如何建立正确心智模型

对你最有用的心智模型是：

> **ARIS = 技能编排层 + 多模型协作层 + 项目文件状态层 + 可选宿主平台层**

换成更直白的话：

- 不是“你装一个程序，它自己会科研”
- 而是“你给一个能执行技能的 Agent 安装 ARIS，ARIS 再把科研拆成一连串可执行步骤”

只要你把这个心智模型建立起来，后面看目录、看技能、看日志都会清晰很多。

## 4. 代码库与分支组织详解

这一章回答两个问题：

1. 你的 5 个分支 / worktree 各自承担什么角色。
2. 每个分支内部有哪些关键子目录。

### 4.1 分支级别的职责划分

| worktree / 分支 | 当前目录 | 角色定位 | 你是否需要频繁进入 |
|---|---|---|---|
| `main` | `~/ARIS/Auto-claude-code-research-in-sleep` | 主技能仓库与主文档仓库，ARIS 方法论主体 | 是 |
| `aris-code` | `~/ARIS/aris-code` | Rust 实现的独立 ARIS-Code CLI | 需要理解，但不是你当前唯一入口 |
| `feature/paper-slides` | `~/ARIS/paper-slides` | `paper-slides` 功能分支快照 | 一般不用日常进入 |
| `feature/paper-poster` | `~/ARIS/paper-poster` | `paper-poster` 功能分支快照 | 一般不用日常进入 |
| `personal` | `~/ARIS/personal` | 你的个人文档与定制分支 | 是 |

### 4.2 `main` 分支的顶层目录角色

`Auto-claude-code-research-in-sleep` 的顶层结构如下：

```text
Auto-claude-code-research-in-sleep/
├── assets/
├── docs/
├── mcp-servers/
├── skills/
├── templates/
├── tests/
├── tools/
├── README.md / README_CN.md
├── CONTRIBUTING*.md
├── .env.example
└── ...
```

它们各自的作用：

#### `assets/`

存放 README、文档演示图、示意图、海报/插图示例等静态资源。它们不是运行逻辑的一部分，主要用于：

- 文档展示
- README 图片
- 技能效果预览

#### `docs/`

这是主线说明文档库，内容非常重要。它不是“可有可无的附录”，而是对主 README 的分流与专题展开。主要包括：

- ARIS-Code 独立 CLI 说明入口
- Session Recovery 指南
- Project Files Guide
- Watchdog 监控指南
- Codex + Claude / Gemini 审稿桥接说明
- Trae / Cursor / Antigravity / OpenClaw 适配说明
- 模型混搭与替代组合说明

如果 README 是总导航，那么 `docs/` 就是专题手册区。

#### `mcp-servers/`

这里放的是本地 MCP bridge / server 实现。它们负责把外部能力接进 ARIS：

- `claude-review/`：让 Codex 工作流把本地 Claude CLI 当 reviewer
- `gemini-review/`：让 Codex 工作流把 Gemini API / CLI 当 reviewer
- `llm-chat/`：通用 OpenAI-compatible reviewer 接口
- `minimax-chat/`：MiniMax reviewer 接口
- `feishu-bridge/`：飞书双向交互桥接

它们的共同特征是：

- 不是主工作流本身
- 而是 reviewer / 通信 / 集成能力的传输层

#### `skills/`

这是整个主分支最重要的目录。它包含：

- 主线科研技能
- 可选技能
- Codex 适配技能
- reviewer overlay
- shared references

这是 ARIS 的“工作流编排层”。

#### `templates/`

模板目录，用于：

- 给工作流提供统一输入结构
- 让输出文件结构化
- 支持 compact 模式与 session 恢复

它定义的是“文件契约”。

#### `tests/`

测试目录主要覆盖：

- MCP server 的协议处理逻辑
- watchdog 任务监控逻辑
- reviewer bridge 的请求处理

注意：这些测试主要验证“桥接脚本和辅助逻辑”，不是在测试整个科研流水线是否能自动发 paper。

#### `tools/`

这里是配套脚本区，承担标准化、确定性较强的任务，例如：

- arXiv 拉取
- Semantic Scholar 拉取
- Research Wiki 维护
- Watchdog 守护
- meta-optimize hook 脚本

### 4.3 `aris-code` 分支的顶层目录角色

`aris-code` 分支是一个 Rust workspace，顶层结构如下：

```text
aris-code/
├── crates/
│   ├── api/
│   ├── aris-cli/
│   ├── commands/
│   ├── compat-harness/
│   ├── runtime/
│   └── tools/
├── docs/
├── Cargo.toml
├── Cargo.lock
├── README*.md
└── CHANGELOG.md
```

它们的角色：

#### `crates/api`

封装消息 API、Sse、类型定义、鉴权来源解析等，是和模型 API 交互的底层。

#### `crates/aris-cli`

CLI 主程序入口。这里实现：

- `aris` 命令
- 参数解析
- REPL 循环
- slash 命令处理
- 模型切换
- reviewer 切换
- session 管理
- skills 列表/导出
- meta-optimize 应用逻辑

#### `crates/commands`

集中定义 slash 命令规格，例如：

- `/help`
- `/model`
- `/reviewer`
- `/skills`
- `/permissions`
- `/session`
- `/meta-optimize`

它更像命令清单与命令解析层。

#### `crates/compat-harness`

这个 crate 的职责是做兼容抽取，帮助从上游参考实现里提取命令/工具清单，理解为“兼容层工具箱”即可。

#### `crates/runtime`

这是 ARIS-Code 的运行时核心。负责：

- 系统提示词构建
- 读取 `CLAUDE.md` / `.claude/settings*.json`
- conversation runtime
- session 存储与压缩
- 事件日志 sink
- MCP client / stdio transport
- hooks、permissions、sandbox、usage 统计
- 内置 bundled skills 打包

#### `crates/tools`

这是 ARIS-Code 的工具定义与执行实现，包括：

- `bash`
- `read_file`
- `write_file`
- `edit_file`
- `glob_search`
- `grep_search`
- `WebFetch`
- `WebSearch`
- `TodoWrite`
- `LlmReview`
- `Skill`

注意这里的 `Skill` 和主仓库的 `skills/` 目录是直接关联的。

### 4.4 `paper-slides` 与 `paper-poster` 分支到底怎么看

这两个 worktree 都有和主线非常相似的结构：

- `assets/`
- `docs/`
- `mcp-servers/`
- `skills/`
- `tools/`

你可以把它们理解为：

- 当时功能开发时的分支工作树
- 带有独立 README 的功能主题快照

在你当前快照中，主线已经整合了这两个功能，因此这两个目录不是“必须安装的额外模块”，而更像：

- 历史视角
- 功能演化快照
- 独立分支保留

### 4.5 `personal` 分支的结构

你现在的 `personal` 目录很简单：

```text
personal/
├── configs/
│   └── codex_prompt.md
└── docs/
    ├── ARIS_Setting.md
    └── ARIS_系统详解与全流程使用手册.md
```

推荐你之后继续把这里当成：

- 你的本地使用手册区
- 你的自定义配置说明区
- 你的踩坑记录区
- 你的实验环境和命令备忘区

这样你拉取上游更新时，主线和个人理解就不会混在一起。

## 5. `main` 分支目录与文件体系详解

这一章进入主线目录内部，也就是你真正最常打交道的代码库主体。

### 5.1 主线目录下最重要的文件

#### `README_CN.md` / `README.md`

这是总入口，主要负责：

- 解释 ARIS 方法论
- 给出快速开始
- 概览工作流 1 / 1.5 / 2 / 3 / 4 / M
- 列出技能总览
- 介绍可选集成（GPU、Zotero、Obsidian、Feishu 等）

你第一次上手一定要看，但它解决的是“导航”，不解决“你当前这台机器的全部细节”，所以我现在这份手册才需要存在。

#### `.env.example`

它列出了一些可选环境变量样例，例如：

- `SEMANTIC_SCHOLAR_API_KEY`
- `MINIMAX_API_KEY`
- `LLM_API_KEY`
- `GEMINI_API_KEY`
- `FEISHU_APP_ID`
- `CLAUDE_REVIEW_MODEL`

它的意义不是让你“必须 export 全部”，而是告诉你：

- 这些 reviewer / bridge / 外部服务可以被环境变量驱动
- 哪些功能依赖哪些 key

#### `CONTRIBUTING*.md`

给贡献者看的协作规范。对使用者不是第一优先级，但有助于理解项目维护方式。

### 5.2 `docs/` 目录逐类说明

主线 `docs/` 大致可以分为 7 类。

#### A. 运行与恢复类

- `PROJECT_FILES_GUIDE_CN.md`
- `SESSION_RECOVERY_GUIDE_CN.md`
- `WATCHDOG_GUIDE_CN.md`

这组文档解决：

- 项目状态文件应该怎么组织
- 上下文压缩/换 session 后如何恢复
- 服务器端如何持续监控实验任务

#### B. 独立 CLI / 代码实现说明

- `ARIS-Code-README_CN.md`

这个文件本身在主线只起“跳转入口”作用，真正内容在 `aris-code` 分支。

#### C. reviewer / 模型桥接类

- `CODEX_CLAUDE_REVIEW_GUIDE_CN.md`
- `CODEX_GEMINI_REVIEW_GUIDE_CN.md`
- `LLM_API_MIX_MATCH_GUIDE.md`
- `MiniMax-GLM-Configuration.md`

这组文档回答：

- 如果执行者不是 Claude，而是 Codex，怎么跑
- 如果 reviewer 不是 Codex，而是 Claude / Gemini / GLM / MiniMax，怎么接
- 模型角色如何混搭

#### D. 宿主平台适配类

- `CURSOR_ADAPTATION.md`
- `TRAE_ARIS_RUNBOOK_CN.md`
- `ANTIGRAVITY_ADAPTATION_CN.md`
- `OPENCLAW_ADAPTATION.md`
- `MODELSCOPE_GUIDE.md`

这组文档说明：

- 不是只有 Claude Code 能用 ARIS
- 只要宿主支持 skill / MCP / 文件操作，ARIS 方法论就能迁移

#### E. 示例与模板说明类

- `NARRATIVE_REPORT_EXAMPLE.md`

帮助你理解工作流 3 输入应该长什么样。

#### F. 图像资源

- logo、banner、screenshot 等

这类文件主要给 README 用。

### 5.3 `mcp-servers/` 目录逐个说明

#### `mcp-servers/claude-review/`

作用：

- 保持 Codex 作为执行者
- 让本地 Claude CLI 作为 reviewer
- 暴露 `review` / `review_reply`
- 以及长任务异步版本 `review_start` / `review_status`

使用场景：

- 你想走 Codex-first 工作流
- 但又想让 Claude 审稿

#### `mcp-servers/gemini-review/`

作用：

- 保持 Codex 作为执行者
- 让 Gemini 作为 reviewer
- 默认 direct Gemini API
- 也支持长审稿异步状态轮询
- 支持 poster PNG 等本地图像审查

#### `mcp-servers/llm-chat/`

作用：

- 提供一个通用 reviewer MCP server
- 对接任意 OpenAI-compatible Chat Completions API

典型用途：

- 用 GLM / DeepSeek / MiniMax / Kimi 做 reviewer
- 替代 Codex reviewer

依赖很轻，只要：

```text
httpx>=0.27,<1.0
```

#### `mcp-servers/minimax-chat/`

作用：

- 专门对接 MiniMax reviewer

#### `mcp-servers/feishu-bridge/`

作用：

- 飞书双向交互桥接
- 支持审批 / 回复 / 长连接交互

依赖：

```text
lark-oapi
```

### 5.4 `tools/` 目录逐个说明

#### `tools/arxiv_fetch.py`

给 `/arxiv` 和 `/research-lit` 提供：

- arXiv 搜索
- arXiv PDF 下载
- 结构化元数据输出

#### `tools/semantic_scholar_fetch.py`

给 `/semantic-scholar` 和部分 `/research-lit` 扩展提供：

- 已发表 venue 论文搜索
- citation count / venue / TLDR / external IDs 等结构化信息

它弥补了 arXiv 只擅长 preprint 的缺口。

#### `tools/research_wiki.py`

Research Wiki 的实用脚本，负责：

- 初始化 wiki
- slug 生成
- edge 维护
- query_pack 重建
- stats / log 维护

#### `tools/watchdog.py`

服务器端长期守护脚本，用来监控：

- 训练 session 是否还活着
- GPU 是否空闲
- 下载是否卡住

它不是主工作流的一部分，但对跨夜实验非常重要。

#### `tools/meta_opt/`

包含：

- `log_event.sh`
- `check_ready.sh`

配合 `templates/claude-hooks/meta_logging.json` 使用，用于：

- 被动记录 skill 调用与工具调用
- 达到阈值后提醒你运行 `/meta-optimize`

### 5.5 `templates/` 目录逐个说明

这些模板是 ARIS 文件协议的标准输入输出。

#### 输入模板

- `RESEARCH_BRIEF_TEMPLATE.md`
  - 给工作流 1 / 全流程的详细研究简报
- `EXPERIMENT_PLAN_TEMPLATE.md`
  - 给工作流 1.5 的实验规划模板
- `NARRATIVE_REPORT_TEMPLATE.md`
  - 给工作流 3 的研究叙事模板
- `PAPER_PLAN_TEMPLATE.md`
  - 跳过 paper-plan 阶段时可直接提供的大纲模板
- `RESEARCH_CONTRACT_TEMPLATE.md`
  - 当前 idea 的聚焦合同文档模板

#### compact / 持久化模板

- `IDEA_CANDIDATES_TEMPLATE.md`
- `EXPERIMENT_LOG_TEMPLATE.md`
- `FINDINGS_TEMPLATE.md`
- `RESEARCH_CONTRACT_TEMPLATE.md`

#### 钩子模板

- `templates/claude-hooks/meta_logging.json`
  - 用于给 Claude Code 安装被动日志 hooks

### 5.6 `tests/` 目录说明

测试集中验证的对象主要是：

- `feishu-bridge` 的消息和回复轮询逻辑
- `llm-chat` 的 JSON-RPC 与 fallback 逻辑
- `minimax-chat` 的模型与 temperature 行为
- `watchdog.py` 的任务注册与状态判断

因此你可以把 `tests/` 看作：

- 对“桥接层、基础设施层”的可靠性验证
- 而不是“科研质量评估”

### 5.7 主线仓库最关键的状态文件体系

这是 ARIS 真正的“数据流核心”。根据 `PROJECT_FILES_GUIDE_CN.md` 与 `SESSION_RECOVERY_GUIDE_CN.md`，最重要的是这几组文件。

#### 项目总状态

- `CLAUDE.md`
  - 项目约束、环境、Pipeline Status、GPU 配置、下一步

#### idea 流

- `IDEA_REPORT.md`
  - 全量 brainstorm + pilot + 筛选结果
- `IDEA_CANDIDATES.md`
  - 精简版候选池
- `docs/research_contract.md`
  - 当前选中的 idea 聚焦文档

#### 实验流

- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `findings.md`

#### review 流

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- 可选 `REVIEWER_MEMORY.md`

#### 写作流

- `PAPER_PLAN.md`
- `figures/`
- `paper/`
- `PAPER_IMPROVEMENT_LOG.md`

#### 长期知识库

- `research-wiki/`

如果你不理解这些文件的关系，ARIS 看起来就像“很多 prompt 在乱飞”；理解之后，你会发现它其实是一个非常明确的文件驱动系统。

## 6. Skill 体系详解

Skill 是 ARIS 的核心。你如果不理解 skill，就不会真正理解 ARIS。

### 6.1 Skill 的本质

在主线仓库里，一个 skill 通常长这样：

```text
skills/<skill-name>/
└── SKILL.md
```

有些 skill 还会附带资源文件，例如 `.py`、`.sh`、`references/` 或 `templates/`。一个 `SKILL.md` 一般包含：

- `name`
- `description`
- `argument-hint`
- `allowed-tools`
- 具体工作流说明
- 默认参数
- 输入输出文件规范
- 失败处理规则
- 与其他 skill 的组合方式

### 6.2 你应该如何理解“调用 skill”

调用 skill 不是“执行一个 shell 脚本”，而是：

1. 宿主 Agent 识别到你请求某个 skill。
2. 宿主读取 `SKILL.md` 内容。
3. 宿主把 `SKILL.md` 当作当前任务的操作手册。
4. 宿主按照手册去调用 Bash / Read / Write / MCP / 子 skill。

因此，skill 更像：

- 高级操作规范
- 工作流模板
- 带文件契约的执行说明书

而不是传统软件里的“函数”。

### 6.3 主线 `skills/` 目录里的内容分层

主线 `skills/` 中既有“真正的科研工作流技能”，也有“Codex 适配技能包”和“overlay”。你应当这样区分：

- 主线技能：给 Claude Code 等主路径使用。
- `skills-codex/`：给 Codex CLI 使用的适配版技能镜像。
- `skills-codex-claude-review/`：Codex 执行、Claude 审稿的 reviewer overlay。
- `skills-codex-gemini-review/`：Codex 执行、Gemini 审稿的 reviewer overlay。
- `shared-references/`：写作与审稿共享参考资料。

### 6.4 主线科研技能分组总览

#### A. 全流程总控

| skill | 作用 |
|---|---|
| `research-pipeline` | 从找方向到实验到自动 review 到写论文的总工作流 |

#### B. 工作流 1：找 idea / 调研 / 精炼

| skill | 作用 |
|---|---|
| `research-lit` | 多源文献综述入口，优先查 Zotero / Obsidian / 本地 PDF，再查 web |
| `arxiv` | 搜索、下载、总结 arXiv 论文 |
| `semantic-scholar` | 查已发表 venue 论文 |
| `idea-creator` | brainstorm 8-12 个研究 idea，筛选并 pilot |
| `novelty-check` | 做查新验证 |
| `research-review` | 让外部 reviewer 深度评议研究方案 |
| `research-refine` | 把模糊 idea 打磨成问题锚点清晰的方法方案 |
| `experiment-plan` | 产出 claim-driven 实验路线图 |
| `research-refine-pipeline` | 将 `research-refine` + `experiment-plan` 串成一体 |
| `idea-discovery` | 工作流 1 总入口 |
| `idea-discovery-robot` | 机器人/具身智能版的 idea-discovery |
| `comm-lit-review` | 通信/无线领域定制调研 |
| `research-wiki` | 持久化研究知识库 |

#### C. 工作流 1.5：实验桥接与执行

| skill | 作用 |
|---|---|
| `experiment-bridge` | 从实验计划到代码实现、部署、结果回收的桥接总流程 |
| `run-experiment` | 本地 / SSH 远程 / Vast.ai / Modal 上部署实验 |
| `monitor-experiment` | 检查实验进度、日志和结果 |
| `training-check` | 用 W&B 指标做训练健康检查 |
| `ablation-planner` | 主结果出来后自动设计消融 |
| `result-to-claim` | 判断结果究竟支持什么 claim |
| `analyze-results` | 结果分析与汇总 |
| `vast-gpu` | Vast.ai GPU 租用与管理 |
| `serverless-modal` | Modal serverless GPU 工作流 |

#### D. 工作流 2：自动 review 循环

| skill | 作用 |
|---|---|
| `auto-review-loop` | 默认多轮外部审稿循环 |
| `auto-review-loop-llm` | 用通用 LLM API 做 reviewer 的版本 |
| `auto-review-loop-minimax` | 用 MiniMax 做 reviewer 的版本 |

#### E. 工作流 3：论文生成

| skill | 作用 |
|---|---|
| `paper-writing` | 工作流 3 总控 |
| `paper-plan` | 写论文大纲与 claims-evidence matrix |
| `paper-figure` | 从实验结果生成图表 |
| `paper-illustration` | 用 Gemini 生成架构图 / 方法图 |
| `paper-write` | 生成 LaTeX 论文正文 |
| `paper-compile` | 编译 LaTeX，修复编译错误 |
| `auto-paper-improvement-loop` | 两轮论文审稿与修正 |
| `mermaid-diagram` | 生成 Mermaid 图 |
| `pixel-art` | 生成像素风 SVG 图，可用于 README 或展示 |

#### F. 工作流 4：投稿后回复

| skill | 作用 |
|---|---|
| `rebuttal` | 审稿意见解析、策略、草稿、安全检查、压力测试、定稿 |

#### G. 展示与衍生产出

| skill | 作用 |
|---|---|
| `paper-slides` | 论文生成演讲幻灯片、speaker notes、talk script、Q&A |
| `paper-poster` | 论文生成会议海报（PDF + PPTX + SVG） |
| `grant-proposal` | 从 idea 生成基金申请书 |

#### H. 理论与专项辅助

| skill | 作用 |
|---|---|
| `proof-writer` | 严格证明撰写 |
| `formula-derivation` | 理论公式推导结构化整理 |
| `dse-loop` | 设计空间探索 |
| `system-profile` | 系统性能分析 |

#### I. 运维与通知

| skill | 作用 |
|---|---|
| `feishu-notify` | 飞书通知 |
| `meta-optimize` | 基于日志分析优化 ARIS 自己 |

### 6.5 最关键的几个 skill，应该怎么理解

#### `research-pipeline`

你可以把它当作“总导演”。它不是亲自做每一件事，而是调度工作流 1、实验实现、工作流 2，以及必要时接工作流 3。适合方向比较明确、希望尽量自动、愿意让系统跨夜运行的情况。

#### `idea-discovery`

这是“找 idea 的全流程导演”，把 `research-lit`、`idea-creator`、`novelty-check`、`research-review`、`research-refine-pipeline` 串起来。适合没有现成 idea、只有研究方向时使用。

#### `experiment-bridge`

这是 ARIS 中非常关键、但最容易被忽略的 skill。它负责从“文字计划”进入“真实代码与实验”，包括解析 `EXPERIMENT_PLAN.md`、阅读已有项目代码或 `base repo`、写实验脚本、调 reviewer 做代码审查、启动 sanity run、自动 debug、部署真正实验、收第一波结果。

#### `auto-review-loop`

这是 ARIS 最具代表性的 skill。它体现了 ARIS 的核心哲学：reviewer 先打分，列弱点，executor 去修，再跑实验，再审，直到达到阈值或到达最大轮数。同时它还负责 `REVIEW_STATE.json` 断点续跑、`AUTO_REVIEW.md` 全量记录，以及可选的 `REVIEWER_MEMORY.md` / `findings.md`。

#### `paper-writing`

它是工作流 3 的导演，串起来 `paper-plan`、`paper-figure`、`paper-write`、`paper-compile`、`auto-paper-improvement-loop`，适合在“实验和叙事已经基本成形”之后启动。

#### `research-wiki`

它不是一次性 workflow，而是长期知识库能力。如果你要长期做一条研究线，这个 skill 的价值非常大，因为它能把读过的 paper、试过的 idea、失败实验、claim 状态都变成可持续记忆，而不是下次重来。

### 6.6 skill 的输入输出为什么这么重要

ARIS 不是靠“隐式理解上下文”来协作，而是靠显式文件协作。举个例子：

- `idea-discovery` 输出 `IDEA_REPORT.md`
- `research-refine-pipeline` 输出 `FINAL_PROPOSAL.md` 和 `EXPERIMENT_PLAN.md`
- `experiment-bridge` 输出实验结果摘要与 `EXPERIMENT_LOG.md`
- `auto-review-loop` 输出 `AUTO_REVIEW.md`
- `paper-writing` 再读取这些内容去生成论文

这就意味着：

- 你随时可以中断
- 你可以换新 session
- 你可以人工插入修改
- 下游 skill 不必依赖“记住上文”

### 6.7 `skills-codex/` 是什么

`skills/skills-codex/README_CN.md` 说明得很清楚：它是一个面向 Codex 的技能包目录，和主线 `skills/` 尽量对齐，但做了 Codex 适配。

其核心特征是：

- 安装位置是 `~/.codex/skills/`
- 保留主线科研技能的同名同步集
- 对技能中的控制型调用方式做 Codex 化适配
- 不负责安装环境、MCP、依赖，只负责技能文件本身

你可以把它理解为：主线 skill 的 Codex 版本镜像。

## 7. `aris-code` 分支详解：独立 CLI 如何实现 ARIS

如果主线仓库回答的是“ARIS 的工作流是什么”，那 `aris-code` 回答的是“把这些工作流做成独立 CLI 时，系统内部怎么实现”。

### 7.1 `aris-code` 的定位

`aris-code` 不是另一个无关项目，而是 ARIS 的产品化运行时。它提供：

- `aris` 命令
- 独立 REPL
- 首次配置向导
- executor / reviewer 模型切换
- slash 命令
- session 管理
- skill 发现与导出
- reviewer 工具 `LlmReview`
- memory 与 meta logging

### 7.2 `aris-code` 的总体模块关系

可以把它看成下面这条链：

```text
aris-cli
→ runtime
→ api / tools / commands
→ bundled skills + user skills
→ 模型 API / reviewer API / 本地会话
```

### 7.3 `crates/aris-cli` 里真正做了什么

根据源码，`crates/aris-cli/src/main.rs` 是主入口，它负责：

- 读取保存的 ARIS 配置
- 解析命令行参数
- 判断进入 REPL、单次 prompt、resume、setup、doctor 等模式
- 创建运行时
- 保存和恢复 session
- 处理 slash 命令

它内置的 slash 命令包括：

- `/help`
- `/status`
- `/compact`
- `/model`
- `/reviewer`
- `/setup`
- `/plan`
- `/tasks`
- `/skills`
- `/permissions`
- `/clear`
- `/cost`
- `/resume`
- `/config`
- `/memory`
- `/init`
- `/diff`
- `/version`
- `/export`
- `/session`
- `/meta-optimize`

而且还有一个很重要的行为：

> 当你输入未知 slash 命令时，它会尝试把这个命令当成 skill 名称解析。

也就是说，在 ARIS-Code CLI 中：

```text
/research-pipeline ...
```

本质上会走“把 `/research-pipeline` 当作 skill 调用”的路径。

### 7.4 `crates/runtime` 里最重要的机制

`runtime` crate 是 ARIS-Code 的骨架，最重要的事情有 5 个：

1. 构建系统提示词：把环境上下文、当前目录、日期、Git 状态、`CLAUDE.md` / `.claude` 指令文件、已发现的可用 skills、运行时配置拼到系统提示里。
2. 发现 instruction files：从当前目录向上查找 `CLAUDE.md`、`CLAUDE.local.md`、`.claude/CLAUDE.md`、`.claude/instructions.md`。
3. 处理 session 与 compaction：`session.rs` 负责 session 存盘、原子写入、轮转归档，`conversation.rs` 负责工具调用和自动压缩。
4. 事件日志：`event_sink.rs` 能把 tool call、skill invoke、slash command、session start/end 写成 JSONL。
5. skill 清单注入：自动搜索可用 skills 并注入系统提示。

### 7.5 `crates/tools` 里最关键的工具

ARIS-Code 的工具清单里，最关键的是 3 个：

#### `Skill`

负责读取 skill 文件。它按下面优先级搜索：

1. `~/.config/aris/skills/`
2. `~/.claude/skills/`
3. 项目级 `.claude/skills/`
4. binary 内置 bundled skills

这说明：

- 你可以在 ARIS-Code 里沿用 `~/.claude/skills/`
- 也可以导出到 `~/.config/aris/skills/` 做 ARIS-Code 专属覆盖

#### `LlmReview`

这是 reviewer 工具。它会把内容发送给外部 reviewer 模型，用于研究审稿、方法批判、论文批判和自动 review loop。所以在 ARIS-Code CLI 里，reviewer 已经被工具化了。

#### `TodoWrite`

它负责持久化任务列表，让 ARIS-Code 能维持结构化任务状态。

### 7.6 `aris-code` 的配置文件在哪里

根据 `crates/aris-cli/src/config.rs`：

- 主配置文件：`~/.config/aris/config.json`
- 用户 skill 覆盖目录：`~/.config/aris/skills/`
- memories 目录：`~/.config/aris/memories/`
- meta proposal / backup / log 也在 `~/.config/aris/` 下

配置主要保存：

- executor provider / API key / base url / model
- reviewer provider / API key / model
- language
- meta logging level

### 7.7 `aris-code` 的会话文件存到哪里

当前项目里的 managed session 目录是：

```text
<cwd>/.claude/sessions/
```

也就是说，ARIS-Code 在项目级别维护 session 文件，而不是全局塞在一个地方。

### 7.8 `aris-code` 对你意味着什么

你现在理解主线仓库就够开始使用了，但 `aris-code` 对你有两个额外价值：

1. 它让你看清“ARIS 真正产品化后需要哪些运行时能力”
2. 它提供了一条未来不完全依赖 Claude Code 的使用路径

如果你以后不想把 ARIS 绑死在 Claude Code 上，`aris-code` 分支是最值得深入研究的实现。

## 8. 运行 ARIS 需要的工具、环境与账号

这一节回答的是最实际的问题：你究竟需要先装什么，哪些是必须的，哪些是可选的，推荐走哪条路径。

### 8.1 先说结论：你最推荐采用哪条使用路径

对于你当前这套仓库，我最推荐的实际使用顺序是：

1. 使用主线仓库 `Auto-claude-code-research-in-sleep` 作为“技能来源和文档源”。
2. 使用 Claude Code 作为主要宿主 Agent。
3. 使用 Codex MCP 作为 reviewer / 外部强审稿模型通道。
4. 在一个独立的研究项目目录里运行 ARIS，而不是直接在 `~/ARIS/Auto-claude-code-research-in-sleep` 仓库里做研究。
5. 如果以后你希望脱离 Claude Code，再考虑研究 `aris-code` 分支。

也就是说，最实用的主路径是：

```text
ARIS 主线仓库（提供 skills）
→ 安装到 ~/.claude/skills/
→ Claude Code 读取 skill
→ review-heavy 步骤通过 Codex MCP 调 GPT-5.4
→ 在你的研究项目目录中执行整套科研流水线
```

### 8.2 必须具备的最小组件

如果你只想先把 ARIS 跑起来，最少需要下面几样：

| 组件 | 是否必须 | 作用 |
|---|---|---|
| Git | 是 | clone 仓库、同步技能 |
| Claude Code | 是（主线路径） | 作为宿主 Agent，负责读取 skill 并执行 |
| `~/.claude/skills/` | 是 | 让 Claude Code 能发现 ARIS skills |
| Node.js / npm | 是 | 安装 Codex CLI 与部分 MCP 工具 |
| `@openai/codex` | 强烈建议 | 作为 reviewer MCP，支撑很多 review 类 skill |
| OpenAI / Codex 可用账号 | 强烈建议 | 给 reviewer 用 |
| Python 3 | 建议 | 运行 `tools/` 里的辅助脚本、文献工具、wiki 工具 |

### 8.3 推荐环境矩阵：你应如何理解不同“宿主”

ARIS 不是一个单独的 daemon，而是“技能包 + 文档约定 + 工具脚本 + 宿主 Agent”的组合。因此你要先分清谁是宿主：

| 路径 | 宿主 | 适合你吗 | 说明 |
|---|---|---|---|
| 主线默认路径 | Claude Code | 是，最推荐 | 文档最完整，示例最全，社区主要围绕这一条路径 |
| Codex 适配路径 | Codex CLI | 可选 | 通过 `skills/skills-codex/` 安装 Codex 版技能 |
| 独立产品化路径 | `aris-code` CLI | 进阶可选 | Rust 实现，更像独立产品 |
| Trae IDE 路径 | Trae | 可选 | 有专门 runbook，但不是你当前最优先的入口 |

你当前最应该掌握的是第一条：Claude Code 主线使用法。

### 8.4 目录层面的准备：技能源与实际研究目录要分开

请牢记这一点：

- `~/ARIS/Auto-claude-code-research-in-sleep` 主要是 ARIS 系统本身。
- 你的真实科研项目应该在另一个目录中进行。

例如你之后真正做研究时，推荐像这样：

```text
/data_3/wly/ARIS/Auto-claude-code-research-in-sleep    ← ARIS 系统仓库
/data_3/wly/research/ddlm-gap                          ← 你某个具体研究项目
```

原因很简单：

- ARIS 仓库里放的是 skill、模板、文档、辅助脚本。
- 研究项目目录里放的是你的代码、实验结果、论文、日志和状态文件。
- 这样 session、`CLAUDE.md`、`AUTO_REVIEW.md`、`paper/` 等项目文件不会污染 ARIS 仓库本身。

### 8.5 Claude Code 主线路径的安装步骤

这是你最应该先完成的一套安装。

#### 第一步：把 ARIS skills 安装到 Claude Code 的全局技能目录

```bash
cd /data_3/wly/ARIS/Auto-claude-code-research-in-sleep
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/
```

这一步的含义是：

- 仓库里的 `skills/` 是“源代码”
- `~/.claude/skills/` 是 Claude Code 真正会去扫描的“运行目录”

之后如果主线仓库更新了 skill，你需要重新同步一次。

如果你不想覆盖本地已经改过的 skill，可以用：

```bash
cp -rn skills/* ~/.claude/skills/
```

#### 第二步：安装 Codex CLI，并把它作为 Claude Code 的 MCP server

```bash
npm install -g @openai/codex
codex setup
claude mcp add codex -s user -- codex mcp-server
```

这三步分别在做：

1. 安装 Codex CLI。
2. 初始化 `~/.codex/config.toml`，并选择模型。
3. 向 Claude Code 注册一个名为 `codex` 的 MCP server。

对于 ARIS，`codex setup` 时最重要的是把模型设成推荐值，例如：

```toml
model = "gpt-5.4"
```

因为很多 review-heavy skill 默认假设外部 reviewer 足够强。

#### 第三步：确认 Claude Code 能看到技能

启动 Claude Code 之后，你应该能直接输入：

```text
/idea-discovery "测试主题"
```

如果技能安装正常，Claude Code 会把它当成一个 skill 来执行，而不是把它当成普通聊天文本。

### 8.6 如果你要走 Codex CLI 路径，安装的是哪套技能

如果你不是让 Claude Code 做宿主，而是直接使用 Codex CLI，则对应的是：

- 主适配目录：`skills/skills-codex/`
- 审稿 overlay：`skills/skills-codex-claude-review/`
- 审稿 overlay：`skills/skills-codex-gemini-review/`

通常安装到：

```text
~/.codex/skills/
```

你可以把它理解为：

- `skills/` 是 Claude Code 版主技能
- `skills/skills-codex/` 是 Codex 版主技能
- `skills/skills-codex-claude-review/` 和 `skills/skills-codex-gemini-review/` 是 reviewer 行为替换层

### 8.7 `aris-code` 路径需要的环境

如果未来你要直接研究 `aris-code` 的独立 CLI 路径，还需要额外准备：

- Rust toolchain
- Cargo
- `~/.config/aris/config.json`
- 可用的 executor model provider
- 可用的 reviewer model provider

但这不是你现在上手 ARIS 的必要前置条件。

### 8.8 与模型相关的账号和 API

ARIS 不是只用一个模型。它通常是“执行模型”和“审稿模型”分工协作。

#### 执行模型

执行模型负责：

- 读写文件
- 理解项目上下文
- 按 skill 工作流一步步执行
- 调 shell、MCP、脚本、Git、LaTeX 等工具

在你当前推荐路径里，执行模型就是 Claude Code 背后的 Claude。

#### 审稿模型

审稿模型负责：

- 以 reviewer 视角挑刺
- 做 novelty check
- 做 research review
- 给 auto-review-loop 打分
- 在实验部署前审代码

在你当前推荐路径里，默认最重要的 reviewer 是通过 Codex MCP 调到的 GPT-5.4。

#### 其他可替代 reviewer

主线仓库也支持其他 reviewer 路径，例如：

- Claude review bridge
- Gemini review bridge
- 通用 LLM API
- MiniMax

这些配置主要体现在：

- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/.env.example`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/.env.example)
- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/LLM_API_MIX_MATCH_GUIDE.md`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/LLM_API_MIX_MATCH_GUIDE.md)
- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/CODEX_CLAUDE_REVIEW_GUIDE_CN.md`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/CODEX_CLAUDE_REVIEW_GUIDE_CN.md)
- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/CODEX_GEMINI_REVIEW_GUIDE_CN.md`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/docs/CODEX_GEMINI_REVIEW_GUIDE_CN.md)

### 8.9 `.env.example` 里这些变量分别在服务什么

主线仓库的 `.env.example` 不是“你必须全部填”，而是一个可选外部能力清单。

#### 文献与检索相关

- `SEMANTIC_SCHOLAR_API_KEY`

用于 Semantic Scholar 检索增强。

#### 通用 LLM reviewer 相关

- `LLM_API_KEY`
- `LLM_BASE_URL`
- `LLM_MODEL`
- `LLM_FALLBACK_MODEL`
- `LLM_SERVER_NAME`

用于 `llm-chat` MCP 路径，让 reviewer 不必依赖 Codex。

#### MiniMax reviewer 相关

- `MINIMAX_API_KEY`
- `MINIMAX_BASE_URL`
- `MINIMAX_MODEL`

用于 `auto-review-loop-minimax` 之类技能。

#### Gemini reviewer 相关

- `GEMINI_API_KEY`
- `GOOGLE_API_KEY`
- `GEMINI_REVIEW_SERVER_NAME`
- `GEMINI_REVIEW_MODEL`
- `GEMINI_REVIEW_BACKEND`

用于 Gemini 审稿桥。

#### Claude reviewer bridge 相关

- `CLAUDE_REVIEW_SERVER_NAME`
- `CLAUDE_BIN`
- `CLAUDE_REVIEW_MODEL`
- `CLAUDE_REVIEW_SYSTEM`
- `CLAUDE_REVIEW_TOOLS`

用于本地 Claude 审稿桥。

#### 通知相关

- `FEISHU_APP_ID`
- `FEISHU_APP_SECRET`
- `FEISHU_USER_ID`
- `BRIDGE_PORT`

用于飞书通知与交互桥接。

### 8.10 可选但非常有价值的外围工具

#### Zotero

作用：

- 给 `research-lit` 提供你的个人论文库
- 读取 PDF 标注、高亮、BibTeX

#### Obsidian

作用：

- 给 `research-lit` 提供你自己的研究笔记
- 把“论文原文”扩展成“你自己的理解和历史判断”

#### LaTeX 工具链

如果你要使用 `paper-writing` 产出 PDF，通常还需要：

```bash
sudo apt install texlive-full latexmk poppler-utils
```

如果不装 LaTeX，写作技能仍可以产出 `.tex` 和论文草稿，但可能无法在本地完整编译 PDF。

#### Python 科研环境

如果你的实验涉及 PyTorch、训练脚本、评测脚本，你还需要你自己的项目环境，例如：

- conda
- uv
- pip
- CUDA / cuDNN
- PyTorch
- 你自己的项目依赖

ARIS 不替你定义研究代码依赖，它只负责组织与驱动。

### 8.11 远程 GPU 服务器如何接入 ARIS

ARIS 的核心思路不是内置一个集群系统，而是让宿主 Agent 从项目里的 `CLAUDE.md` 学会你的运行环境。

仓库文档给出的典型写法是，在项目 `CLAUDE.md` 中放类似内容：

```markdown
## 远程服务器

- SSH：`ssh my-gpu-server`
- GPU：4x A100
- Conda 环境：`research`
- 激活：`eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- 代码目录：`/home/user/experiments/`
- 后台运行：`screen -dmS exp0 bash -c '...'`
```

这类信息的作用不是“给人看”，而是让宿主 Agent 在执行 `run-experiment` / `monitor-experiment` 时知道：

- 去哪里 ssh
- 如何激活环境
- 代码同步到哪里
- 用什么后台方式提交
- 日志应该去哪里找

### 8.12 运行前你至少应该完成的检查清单

在真正开始一次科研流水线前，至少确认以下几点：

- `~/.claude/skills/` 已经安装 ARIS skills。
- Claude Code 能识别 `/idea-discovery` 或 `/research-pipeline`。
- Codex MCP 已配置成功。
- `~/.codex/config.toml` 中模型已设为你想要的 reviewer 模型。
- 你的研究项目目录已经单独创建。
- 项目根目录里已经写好 `CLAUDE.md`。
- 如果要跑实验，你已经把 GPU / SSH / conda / 代码目录信息写入 `CLAUDE.md`。
- 如果要写论文，你已经装好 LaTeX。
- 如果要接 Zotero / Obsidian / 飞书 / W&B，相应服务也已准备好。

当这些都具备后，ARIS 才是真正“可用”而不是“仓库已 clone”。

## 9. 你实际应该怎么使用 ARIS

这一节不是讲理念，而是讲你以后每次开一个新研究项目时，实际应该怎么操作。

### 9.1 正确的使用单位不是“仓库”，而是“一个研究项目目录”

ARIS 的正确使用方式不是：

- 进入 `~/ARIS/Auto-claude-code-research-in-sleep`
- 直接在那里开始跑自己的课题

而是：

1. 保留 ARIS 仓库作为系统仓库。
2. 另外创建一个研究项目目录。
3. 在那个项目目录中启动 Claude Code。
4. 让 Claude Code 借助全局已安装的 ARIS skills 来工作。

例如：

```bash
mkdir -p /data_3/wly/research/ddlm-gap
cd /data_3/wly/research/ddlm-gap
claude
```

### 9.2 一个新项目目录里，最少应该有哪些文件

推荐最少具备以下内容：

```text
ddlm-gap/
├── CLAUDE.md
├── RESEARCH_BRIEF.md              ← 可选，但强烈建议
├── docs/
│   └── research_contract.md       ← 后续建议维护
├── src/ 或你的研究代码目录
├── data/                          ← 视情况
├── results/                       ← 视情况
└── paper/                         ← 后续 workflow 3 会生成
```

其中最重要的是两个文件：

- `CLAUDE.md`
- `RESEARCH_BRIEF.md`

### 9.3 `CLAUDE.md` 是什么，为什么它如此关键

`CLAUDE.md` 不是论文文档，而是项目级运行说明书。宿主 Agent 会把它当成长期上下文的一部分。

这里建议你至少写四类信息：

#### A. 项目目标

- 你要研究什么
- 不做什么
- 成功标准是什么

#### B. 环境约束

- 当前机器有没有 GPU
- 是否需要远程服务器
- 远程服务器怎么 ssh
- conda / uv 环境怎么激活

#### C. 代码目录与运行习惯

- 代码应该放在哪
- 后台任务用什么方式提交
- 日志在哪

#### D. 当前状态

- 还在找 idea
- 已经进入实验
- 已经进入写作

你可以写成类似下面这样：

```markdown
# Project: DLM Factorized Gap

## Goal
- 研究离散扩散语言模型中的 factorized gap 问题。
- 目标是形成可投稿到 NeurIPS / ICLR 的完整实验与论文。

## Constraints
- 总预算：200 GPU-hours
- 截止时间：2026-08-15
- 不做超大规模预训练，只做方法改进与中等规模验证

## Remote Server
- SSH：`ssh my-gpu-server`
- GPU：4x A100 80GB
- Conda 环境：`research`
- 激活：`eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- 代码目录：`/home/wly/projects/ddlm-gap`
- 后台运行：`screen -dmS exp bash -c '...'`

## Working Rules
- 所有实验结果写入 `results/`
- 所有表格保存为 CSV/JSON，不只保留终端输出
- 所有重要结论同步回 `docs/research_contract.md`

## Pipeline Status
- Current stage: idea discovery
```

### 9.4 `RESEARCH_BRIEF.md` 应该什么时候写

如果你的课题背景比较复杂，不要只给一句话 prompt。应该在项目根目录先写 `RESEARCH_BRIEF.md`。

这个文件适合写：

- 问题背景
- 你已读过哪些 paper
- 你已经试过什么
- 哪些失败了
- 有哪些算力和时间约束
- 你真正想要的是“新方向”还是“改进已有方法”

ARIS 已经提供模板：

- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_BRIEF_TEMPLATE.md`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_BRIEF_TEMPLATE.md)

你可以照着模板复制一份到你的研究项目根目录中。

### 9.5 你到底是“输入一句命令”，还是“先 @ 一个 skill 文件”

这取决于宿主。

#### 在 Claude Code 里

最典型的是直接输入 slash 风格命令：

```text
/idea-discovery "离散扩散语言模型的 factorized gap"
/experiment-bridge
/auto-review-loop "离散扩散语言模型的 factorized gap"
/paper-writing "NARRATIVE_REPORT.md"
```

Claude Code 会把这些名字映射到安装好的 skill。

#### 在一些不原生支持 slash 技能的宿主里

有时要显式引用 skill 文件，例如：

```text
@skills/auto-review-loop/SKILL.md
使用这个 skill，评审并改进当前项目。
```

这类写法本质上仍然是在“把 skill 当说明书读入上下文”。

#### 在 `aris-code` CLI 里

你可以用 slash 命令，因为未知 slash 命令会被解释为 skill 名称：

```text
/research-pipeline "topic"
```

### 9.6 你发出的命令，ARIS 内部到底发生了什么

以：

```text
/idea-discovery "离散扩散语言模型的 factorized gap"
```

为例，宿主 Agent 内部通常会经历下面的过程：

1. 识别到你要调用 `idea-discovery`。
2. 去 skill 搜索路径中找到对应的 `SKILL.md`。
3. 读取 `SKILL.md` 的目标、参数、工作流和输出契约。
4. 结合当前项目目录中的 `CLAUDE.md`、`RESEARCH_BRIEF.md`、现有代码和已有结果建立上下文。
5. 依次调用子 skill，例如 `research-lit`、`idea-creator`、`novelty-check`、`research-review`、`research-refine-pipeline`。
6. 把中间结果写入项目文件，而不是只在聊天上下文里“记住”。
7. 在某些 checkpoint 等待你的确认，或按默认值继续。

也就是说，你输入的是一句命令，但系统执行的是一条长链。

### 9.7 ARIS 常用调用方式一览

#### 方式 A：最推荐，直接调用顶层 skill

适合你已经知道自己想走哪条工作流。

```text
/idea-discovery "研究方向"
/experiment-bridge
/auto-review-loop "论文主题"
/paper-writing "NARRATIVE_REPORT.md"
/research-pipeline "研究方向"
```

#### 方式 B：带参数覆盖

适合你想改默认行为。

```text
/research-pipeline "研究方向" — AUTO_PROCEED: false, human checkpoint: true
/idea-discovery "研究方向" — sources: zotero, web, arxiv download: true
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — code review: true, wandb: true
/paper-writing "NARRATIVE_REPORT.md" — venue: NeurIPS
```

你要注意几点：

- ARIS 很多参数支持沿调用链向下透传。
- 你在总控 skill 上设置的参数，常常会被子 skill 继续使用。
- 参数名大小写不一定严格统一，但你最好贴近 README / SKILL.md 的写法。

#### 方式 C：分步执行，而不是全自动一把梭

适合你刚上手，或者你想每一步都自己确认。

推荐分步顺序：

1. `/research-wiki init`
2. `/research-lit "topic"`
3. `/idea-creator "topic"`
4. `/novelty-check "candidate idea"`
5. `/research-review "proposal"`
6. `/research-refine`
7. `/experiment-plan`
8. `/experiment-bridge`
9. `/result-to-claim`
10. `/auto-review-loop "topic"`
11. `/paper-writing "NARRATIVE_REPORT.md"`

这种方式更利于你理解 ARIS 每一步到底在做什么。

### 9.8 研究过程中会生成哪些关键文件

你应该把 ARIS 理解成“持续生成项目状态文件”的系统。

常见重要文件包括：

| 文件 | 典型来源 | 作用 |
|---|---|---|
| `RESEARCH_BRIEF.md` | 你手动写 | 课题背景输入 |
| `IDEA_REPORT.md` | `idea-discovery` | 候选 idea 列表、评分、排名 |
| `FINAL_PROPOSAL.md` | `research-refine-pipeline` | 最终方法提案 |
| `EXPERIMENT_PLAN.md` | `experiment-plan` | claim-driven 实验设计 |
| `docs/research_contract.md` | 你或 skill 维护 | 当前已选 idea 的单页工作契约 |
| `EXPERIMENT_LOG.md` | 实验相关技能 | 实验摘要、命令、结果 |
| `AUTO_REVIEW.md` | `auto-review-loop` | 每轮 reviewer 反馈与修复记录 |
| `REVIEW_STATE.json` | `auto-review-loop` | review 断点续跑状态 |
| `NARRATIVE_REPORT.md` | 你或 skill 汇总 | 研究叙事输入 |
| `paper/` | `paper-writing` | LaTeX、图表、PDF |
| `research-wiki/` | `research-wiki` | 长期知识库 |

### 9.9 `docs/research_contract.md` 为什么强烈建议你维护

ARIS 仓库模板里专门提供了：

- [`/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_CONTRACT_TEMPLATE.md`](/data_3/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_CONTRACT_TEMPLATE.md)

它的价值非常大，因为它把“最终选择的 idea”从冗长的 `IDEA_REPORT.md` 里抽出来，变成当前 session 和未来恢复 session 时最值得读的一页纸。

这里建议你在选定 idea 之后立刻创建：

```text
docs/research_contract.md
```

然后持续更新：

- 核心 claim
- 方法摘要
- 当前最好的结果
- 关键 decision
- 已完成状态

这会显著提高 session 恢复后的稳定性。

### 9.10 什么时候用 `research-pipeline`，什么时候分步跑

#### 用 `research-pipeline` 的情况

- 你已经基本熟悉 ARIS
- 你愿意接受更高自动化
- 你有相对明确的研究方向
- 你已经把环境和 `CLAUDE.md` 配好了

#### 分步跑的情况

- 你第一次接触 ARIS
- 你想理解每个 skill 的职责
- 你不想让系统自动花 GPU
- 你希望人工挑选 idea

对你目前来说，我建议先分步跑 1 到 2 次，再考虑上 `research-pipeline`。

### 9.11 `research-wiki` 应该在什么时候启用

如果你只是体验一遍流程，可以先不启用。

如果你准备长期做某条研究线，建议一开始就在项目里执行：

```text
/research-wiki init
```

初始化后，项目中会出现 `research-wiki/` 目录。后续它的价值体现在：

- `research-lit` 可以把读到的 paper 入库
- `idea-creator` 会读取历史 idea 和记忆
- `result-to-claim` 会更新 claim 的支持状态
- 失败思路也会留下，避免重复踩坑

### 9.12 如果中断了，下次怎么继续

ARIS 的恢复依赖的是“项目文件”，不是聊天记忆。

你下次继续时，正确做法通常是：

1. 回到项目目录。
2. 检查 `CLAUDE.md`、`docs/research_contract.md`、`AUTO_REVIEW.md`、`REVIEW_STATE.json`、`paper/` 等当前状态文件。
3. 启动新 session。
4. 直接告诉宿主继续下一步，或重新调用上一次的 skill。

例如：

```text
请先阅读 CLAUDE.md、docs/research_contract.md、AUTO_REVIEW.md 和 REVIEW_STATE.json，然后继续 auto-review-loop 的下一轮。
```

或者：

```text
/auto-review-loop "当前项目主题" — human checkpoint: true
```

### 9.13 你当前最推荐的上手顺序

结合你目前“还不熟悉 ARIS”的状态，我建议你第一次实际体验按下面顺序来：

1. 安装 `skills/*` 到 `~/.claude/skills/`
2. 配好 Codex MCP
3. 新建一个独立研究项目目录
4. 写 `CLAUDE.md`
5. 写一个简要的 `RESEARCH_BRIEF.md`
6. 先跑一次 `/idea-discovery`
7. 阅读 `IDEA_REPORT.md`、`FINAL_PROPOSAL.md`、`EXPERIMENT_PLAN.md`
8. 手动创建并维护 `docs/research_contract.md`
9. 再跑 `/experiment-bridge`
10. 结果初步出来后跑 `/result-to-claim`
11. 进入 `/auto-review-loop`
12. 最后再跑 `/paper-writing`

这条路径最容易学懂，也最不容易失控。

## 10. 一次完整自动科研的模拟演练

这一节非常重要。它不是告诉你“理论上可以怎么做”，而是模拟你第一次真正用 ARIS 做一项研究时，从空目录开始，到产出论文草稿，完整会经历什么。

下面的例子我不实际运行，但会尽量按真实 ARIS 行为来写。

### 10.1 场景设定

假设你要做的题目是：

```text
离散扩散语言模型中的 factorized gap：如何减少训练目标与采样行为之间的不一致
```

假设你的资源情况是：

- 你本机主要负责控制与写作
- 你有一台远程 GPU 服务器
- 你希望最终得到可投稿论文草稿
- 你希望先体验完整流程，但不要让系统一开始就完全失控

我们假定项目目录为：

```text
/data_3/wly/research/ddlm-gap
```

### 10.2 第 0 步：创建项目目录

你先在普通终端中做：

```bash
mkdir -p /data_3/wly/research/ddlm-gap
cd /data_3/wly/research/ddlm-gap
mkdir -p docs results paper
```

此时目录里还没有真正的科研状态文件。

### 10.3 第 1 步：写 `CLAUDE.md`

你在项目根目录创建 `CLAUDE.md`。建议至少写成这样：

```markdown
# Project: DLM Factorized Gap

## Goal
- 研究离散扩散语言模型中的 factorized gap 问题。
- 目标是构造一个可验证的方法改进，而不是只做现象分析。

## Constraints
- 算力预算：200 GPU-hours
- 截止时间：2026-08-15
- 目标会议：NeurIPS 2026

## Remote Server
- SSH：`ssh my-gpu-server`
- GPU：4x A100 80GB
- Conda 环境：`research`
- 激活：`eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- 代码目录：`/home/wly/projects/ddlm-gap`
- 后台运行：`screen -dmS exp bash -c '...'`

## Working Rules
- 所有实验结果存 `results/`
- 所有关键实验都要追加写入 `EXPERIMENT_LOG.md`
- 所有 claim 的变更都要同步到 `docs/research_contract.md`

## Pipeline Status
- Current stage: idea discovery
```

这一步完成后，宿主 Agent 已经知道：

- 你的课题目标
- 有哪些资源限制
- 如果要跑实验，应该去哪里、怎么激活环境、怎么后台提交

### 10.4 第 2 步：写 `RESEARCH_BRIEF.md`

如果你只打一行 prompt，ARIS 也能工作；但为了更稳定，建议先写简报。

例如：

```markdown
# Research Brief

## Problem Statement
当前离散扩散语言模型在训练目标与推理采样之间存在 factorized gap。
我们怀疑分解式建模导致局部一致但全局采样轨迹失真，从而在长序列生成中出现误差累积。

## Background
- Field: NLP
- Sub-area: discrete diffusion language models
- Key papers I've read: SEDD, D3PM, masked diffusion LM, entropy-based decoding
- What I already tried: 调过 noise schedule，改过 loss reweighting
- What didn't work: 仅调温度和采样步数没有本质改善

## Constraints
- Compute: 200 GPU-hours
- Timeline: 4 months
- Target venue: NeurIPS 2026

## What I'm Looking For
- [x] Improvement on existing method
- [ ] New research direction from scratch

## Domain Knowledge
我更关心目标失配与采样轨迹控制，不想做大规模预训练。

## Non-Goals
不做超大参数量 scaling paper。

## Existing Results
已有初步观察表明：短序列上训练 loss 更低并不一定对应更好采样质量。
```

### 10.5 第 3 步：启动 Claude Code

在项目目录里启动：

```bash
cd /data_3/wly/research/ddlm-gap
claude
```

此时 Claude Code 的上下文里，最关键的项目文件是：

- `CLAUDE.md`
- `RESEARCH_BRIEF.md`

如果你已经启用了 `research-wiki`，还会有：

- `research-wiki/`

### 10.6 第 4 步：先初始化长期记忆（可选但推荐）

你输入：

```text
/research-wiki init
```

宿主 Agent 典型动作：

1. 读取 `research-wiki` skill。
2. 在项目根目录创建 `research-wiki/`。
3. 初始化 wiki 的目录结构与索引。

此时目录可能变成：

```text
ddlm-gap/
├── CLAUDE.md
├── RESEARCH_BRIEF.md
├── docs/
├── paper/
├── results/
└── research-wiki/
```

这一步不是必须，但如果后面你会多轮迭代，它很值。

### 10.7 第 5 步：运行工作流 1，找 idea

你输入：

```text
/idea-discovery "离散扩散语言模型的 factorized gap" — AUTO_PROCEED: false, sources: web, semantic-scholar, arxiv download: true
```

这里我故意加了 `AUTO_PROCEED: false`，因为你第一次使用时最好在选 idea 那一关停一下。

#### 这一命令内部会发生什么

宿主 Agent 典型会做下面这些事：

1. 读取 `idea-discovery/SKILL.md`。
2. 发现它本身是总控 skill，于是继续组织子流程。
3. 读取项目的 `RESEARCH_BRIEF.md`。
4. 调用 `research-lit` 做文献调研。
5. `research-lit` 视配置使用：
   - 本地文档
   - Web 搜索
   - arXiv
   - Semantic Scholar
   - 可选的 Zotero / Obsidian
6. 生成文献摘要与问题空间理解。
7. 调用 `idea-creator` 产生多组候选 idea。
8. 调用 `novelty-check` 排除明显已发表或过于相似的点子。
9. 调用 `research-review` 让 reviewer 从审稿角度质疑这些 idea。
10. 对最优 idea 调用 `research-refine-pipeline`，进一步落成方案与实验计划。

#### 这一阶段会产生哪些关键文件

最重要的是：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`

如果启用了 compact 选项，还可能有：

- `IDEA_CANDIDATES.md`

#### 这一步后你应该看到什么

理想情况下，Agent 会给你一个阶段性总结，例如：

```text
已生成 9 个候选 idea。
其中 2 个在 novelty-check 中被淘汰。
当前排名前 3 的方案已经形成最终 proposal 和实验计划。
请确认是否继续使用排名第一的方案。
```

由于你设了 `AUTO_PROCEED: false`，它通常会在这里等你决定。

### 10.8 第 6 步：你如何阅读并确认 `idea-discovery` 的输出

你此时应该重点看：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`

你要检查的不是文风，而是三件事：

1. 排名第一的 idea 是否真是你想做的。
2. `FINAL_PROPOSAL.md` 是否把方法讲清楚了。
3. `EXPERIMENT_PLAN.md` 的实验是否与核心 claim 对齐。

如果你同意继续，可以在 Claude Code 里回复类似：

```text
选择排名第一的方案继续。请把当前选中的 idea 提炼成 docs/research_contract.md，并进入实验桥接。
```

### 10.9 第 7 步：创建 `docs/research_contract.md`

严格说，这一步可以由你手动做，也可以要求 Agent 根据模板创建。

你可以直接说：

```text
请根据 IDEA_REPORT.md 和 refine-logs/FINAL_PROPOSAL.md 创建 docs/research_contract.md，只保留当前选中方案。
```

宿主 Agent 典型动作：

1. 读取模板逻辑。
2. 从 `IDEA_REPORT.md` 中抽出被选中的 idea。
3. 从 `FINAL_PROPOSAL.md` 中抽出方法摘要与 claim。
4. 写入 `docs/research_contract.md`。

这一步的价值在于：后面恢复 session 时，不必再把整个 `IDEA_REPORT.md` 全部塞回上下文。

### 10.10 第 8 步：运行工作流 1.5，把计划桥接成真实实验

你输入：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — code review: true, wandb: true
```

#### 这一命令内部典型会发生什么

1. 读取 `experiment-bridge/SKILL.md`。
2. 读取：
   - `refine-logs/EXPERIMENT_PLAN.md`
   - `refine-logs/EXPERIMENT_TRACKER.md`
   - `refine-logs/FINAL_PROPOSAL.md`
   - `docs/research_contract.md`
3. 解析出：
   - 要实现哪些模块
   - 要跑哪些实验
   - 成功判据是什么
4. 检查项目里是否已有代码；如果有 `base repo`，则基于它改。
5. 让执行模型写代码、写脚本、写配置。
6. 如果 `code review: true`，在正式耗费 GPU 之前调用 reviewer 审代码。
7. 先做 sanity run。
8. 再按 `CLAUDE.md` 中的服务器配置部署实验。
9. 收集初始结果，更新 tracker 和实验日志。

#### 这一阶段常见会写入什么

- 你的项目代码目录中的实现代码
- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `results/` 下的中间产物
- 可选的 W&B 运行记录

如果一切顺利，你通常会得到类似总结：

```text
已完成最小可运行版本实现。
2 个 sanity runs 成功。
3 个主实验已提交到远程服务器。
EXPERIMENT_TRACKER.md 已更新。
```

### 10.11 第 9 步：实验运行期间，如何监控

你可以输入：

```text
/monitor-experiment
```

或更明确地说：

```text
请检查 refine-logs/EXPERIMENT_TRACKER.md、results/ 和远程日志，汇总当前实验状态。
```

宿主 Agent 常见动作：

1. 读取 tracker。
2. ssh 到服务器检查后台任务。
3. 看日志文件是否报错。
4. 若启用 W&B，则抓训练曲线。
5. 更新 `refine-logs/EXPERIMENT_TRACKER.md`。
6. 将已完成实验追加写入 `EXPERIMENT_LOG.md`。

这里你要特别注意：

- `EXPERIMENT_TRACKER.md` 偏“任务看板”
- `EXPERIMENT_LOG.md` 偏“永久实验档案”

两者不要混淆。

### 10.12 第 10 步：结果出来后，把结果翻译成 claim

你输入：

```text
/result-to-claim
```

这一 skill 的作用不是跑实验，而是解释实验。

它会重点读取：

- `EXPERIMENT_LOG.md`
- `EXPERIMENT_TRACKER.md`
- 当前方法描述

然后判断：

- 哪些 claim 被支持
- 哪些只被部分支持
- 哪些被否定
- 应该缩小 claim，还是需要新增实验

这一步很关键，因为很多人会把“数字变好了一点”误当成“论点成立”。ARIS 专门把这一步抽成了 skill。

如果项目中已经启用 `research-wiki/`，它还可能把 claim 状态同步进 wiki。

### 10.13 第 11 步：进入工作流 2，自动 review 循环

你输入：

```text
/auto-review-loop "离散扩散语言模型的 factorized gap" — human checkpoint: true, difficulty: hard
```

我建议你第一次把 `human checkpoint` 打开，因为这样每轮结束后它会停下来让你看。

#### 这一命令内部典型会发生什么

1. 检查项目根目录是否已有 `REVIEW_STATE.json`。
2. 如果有，尝试从上次 review 轮次恢复。
3. 读取项目叙事与实验结果：
   - `AUTO_REVIEW.md`（若已有）
   - `NARRATIVE_REPORT.md`（若已有）
   - `EXPERIMENT_LOG.md`
   - `findings.md`（若 compact 模式）
   - 代码与结果目录
4. 让 reviewer 以审稿人视角打分、挑核心弱点。
5. 让执行模型按照 reviewer 建议去补实验、修代码、改叙事。
6. 每轮结束更新：
   - `AUTO_REVIEW.md`
   - `REVIEW_STATE.json`
   - `findings.md`（若 compact）
7. 达到阈值或轮数上限后停止。
8. 最后尝试调用 `result-to-claim`，把结果归纳成结构化 claim。

#### 这一步最关键的两个文件

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`

前者是完整评审历史，后者是断点续跑状态。

#### 这一步的典型人机交互是什么样

一轮 review 完后，宿主可能会给你类似反馈：

```text
Round 1 score: 5.8/10
主要问题：
1. factorized gap 的理论动机不够实证化
2. 缺少与 temperature-only baseline 的系统对比
3. 长序列退化分析还不够

建议修复：
- 新增两个对照实验
- 补一个误差累积可视化
- 缩小主 claim 的表述

是否继续下一轮？
```

如果你设置了 `human checkpoint: true`，你可以回复：

```text
继续，但优先做 temperature baseline，对长序列分析只做 1 个代表性数据集。
```

然后它再继续下一轮。

### 10.14 第 12 步：在 review 循环后整理叙事输入

当工作流 2 结束后，你通常应该有足够材料写 `NARRATIVE_REPORT.md` 了。

你可以让 Agent 帮你整理：

```text
请根据 docs/research_contract.md、EXPERIMENT_LOG.md、AUTO_REVIEW.md 和最新结果，生成 NARRATIVE_REPORT.md。
```

这时它会把散落在各处的：

- 核心问题
- 方法
- 主结果
- 消融
- 局限性
- 图表建议

汇总成适合工作流 3 的输入文件。

### 10.15 第 13 步：进入工作流 3，自动写论文

你输入：

```text
/paper-writing "NARRATIVE_REPORT.md" — venue: NeurIPS
```

#### 这一命令内部会按什么顺序工作

根据 skill 设计，它会串：

```text
/paper-plan
→ /paper-figure
→ /paper-write
→ /paper-compile
→ /auto-paper-improvement-loop
```

#### 它通常会生成哪些结果

- `PAPER_PLAN.md`
- `paper/figures/`
- `paper/main.tex`
- `paper/sections/*.tex`
- `paper/references.bib`
- `paper/main.pdf`
- `paper/PAPER_IMPROVEMENT_LOG.md`

它还可能保留阶段性 PDF：

- `paper/main_round0_original.pdf`
- `paper/main_round1.pdf`
- `paper/main_round2.pdf`

### 10.16 第 14 步：如果需要，再生成 slides 和 poster

论文稿基本成型后，你还可以继续：

```text
/paper-slides "paper/"
/paper-poster "paper/"
```

这两个 skill 属于工作流之外的衍生产出阶段。

### 10.17 第 15 步：如果已经收到了审稿意见，再进入 rebuttal

你可以输入：

```text
/rebuttal "paper/ + reviews" — venue: NeurIPS
```

此时 ARIS 会读取：

- 论文
- reviewer comments
- 既有实验与 claim

然后生成 rebuttal 策略与草稿，必要时还可以衔接新的补实验。

### 10.18 这一整套模拟流程里，目录会怎么演化

在一个比较完整的运行后，你的项目目录大概会变成：

```text
ddlm-gap/
├── CLAUDE.md
├── RESEARCH_BRIEF.md
├── IDEA_REPORT.md
├── IDEA_CANDIDATES.md                  ← 若 compact
├── EXPERIMENT_LOG.md
├── AUTO_REVIEW.md
├── REVIEW_STATE.json
├── NARRATIVE_REPORT.md
├── PAPER_PLAN.md
├── findings.md                         ← 若 compact
├── docs/
│   └── research_contract.md
├── refine-logs/
│   ├── FINAL_PROPOSAL.md
│   ├── EXPERIMENT_PLAN.md
│   └── EXPERIMENT_TRACKER.md
├── research-wiki/
├── results/
├── paper/
│   ├── main.tex
│   ├── main.pdf
│   ├── references.bib
│   ├── sections/
│   ├── figures/
│   └── PAPER_IMPROVEMENT_LOG.md
└── .claude/
    └── sessions/                       ← 视宿主而定
```

### 10.19 你应该如何理解“日志到底存在哪里”

ARIS 没有单一总日志文件，而是多层日志：

#### 项目状态日志

- `IDEA_REPORT.md`
- `EXPERIMENT_LOG.md`
- `AUTO_REVIEW.md`
- `PAPER_IMPROVEMENT_LOG.md`

这些是“给人和 Agent 都能读”的业务日志。

#### 断点状态日志

- `REVIEW_STATE.json`
- `refine-logs/EXPERIMENT_TRACKER.md`

这些是“让流程能继续跑”的状态日志。

#### 会话级日志

如果你走 Claude / ARIS-Code 的 session 管理路径，可能还会有：

- `.claude/sessions/`

#### 元日志

如果启用 meta logging，还可能有：

- `.aris/meta/events.jsonl`
- `~/.aris/meta/events.jsonl`
- `~/.config/aris/meta/events.jsonl`（ARIS-Code 路径）

### 10.20 你在整套流程中真正扮演什么角色

很多人会误以为“自动科研”意味着人完全退出。实际上在 ARIS 中，你最适合扮演的是：

- 设定问题边界的人
- 批准 idea 的人
- 控制算力预算的人
- 审核主 claim 是否可信的人
- 最终决定论文口径的人

ARIS 擅长的是：

- 把流程组织起来
- 把状态写下来
- 自动调用 reviewer
- 让项目可以跨 session 连续推进

### 10.21 第一次体验时的最稳妥版本

如果你想第一次体验，但不想直接全自动到底，我建议你这样执行：

1. `/research-wiki init`
2. `/idea-discovery "topic" — AUTO_PROCEED: false`
3. 人工确认选中的 idea
4. 创建 `docs/research_contract.md`
5. `/experiment-bridge`
6. `/monitor-experiment`
7. `/result-to-claim`
8. `/auto-review-loop "topic" — human checkpoint: true`
9. 生成 `NARRATIVE_REPORT.md`
10. `/paper-writing "NARRATIVE_REPORT.md"`

这条路径比直接一上来 `/research-pipeline` 更适合你当前阶段。

## 11. 常见误区与建议的上手路径

### 11.1 常见误区

#### 误区 1：以为 ARIS 是一个可以直接 `python main.py` 跑起来的单体程序

不是。

ARIS 主线更像：

- 技能系统
- 工作流约定
- 文件契约
- 宿主 Agent 上运行的科研操作体系

所以你找不到一个统一的“入口二进制”是正常的。

#### 误区 2：以为 clone 完仓库就等于已经安装完成

不是。

clone 只是把源码和文档拿到了本地。真正要能用，至少还要：

- 把 skills 同步到宿主能扫描的目录
- 配好 reviewer MCP
- 在独立研究项目目录中启动宿主 Agent

#### 误区 3：以为应该在 ARIS 仓库目录里直接做自己的研究

不建议。

真正的研究应该在单独项目目录里进行。ARIS 仓库主要是技能库和说明书。

#### 误区 4：以为 skill 是 shell 脚本

不是。

skill 的本质是“操作手册 + 文件约定 + 调度说明”，宿主 Agent 读取后再决定如何调用 Bash、MCP、脚本和子 skill。

#### 误区 5：以为 `/research-pipeline` 一条命令就总是最优

不是。

对第一次使用者来说，分步使用通常更好，因为：

- 你能看懂每个阶段的产物
- 你能控制算力消耗
- 你能更早发现方向走偏

#### 误区 6：以为自动 review 只是“让另一个模型点评两句”

不是。

在 ARIS 里，`auto-review-loop` 是一个真正的循环系统：

- reviewer 给分和找问题
- executor 去改代码、补实验、改叙事
- 状态写回文件
- 再进行下一轮

#### 误区 7：以为聊天历史就是唯一上下文

不是。

ARIS 真正依赖的是项目文件：

- `CLAUDE.md`
- `docs/research_contract.md`
- `EXPERIMENT_LOG.md`
- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- `paper/`

这也是它能跨 session 继续工作的根本原因。

#### 误区 8：以为所有工具和集成都必须先装齐

也不是。

你完全可以先用最小路径开始：

- Claude Code
- ARIS skills
- Codex MCP
- 一个简单项目目录

Zotero、Obsidian、W&B、飞书、Gemini、MiniMax 都可以后面逐步加。

### 11.2 建议你的上手路线分成三层

#### 第一层：先学会“看懂文件流”

目标：

- 理解 `CLAUDE.md`
- 理解 `IDEA_REPORT.md`
- 理解 `FINAL_PROPOSAL.md`
- 理解 `EXPERIMENT_PLAN.md`
- 理解 `AUTO_REVIEW.md`

在这一步，你甚至不需要真的跑大实验。

#### 第二层：学会“分步调用 skill”

推荐顺序：

1. `/research-wiki init`
2. `/idea-discovery`
3. 阅读输出
4. 创建 `docs/research_contract.md`
5. `/experiment-bridge`
6. `/monitor-experiment`
7. `/result-to-claim`
8. `/auto-review-loop`
9. `/paper-writing`

当你能理解每一步生成了什么文件，你就算真正入门了。

#### 第三层：再尝试“总控自动化”

当你已经知道：

- 哪些参数会影响行为
- 什么时候该人工 checkpoint
- 你的远程环境写法是否稳定
- 哪些实验最容易失败

这时你再尝试：

```text
/research-pipeline "你的研究方向"
```

才更合适。

### 11.3 针对你当前情况的最优上手法

针对你现在“对 ARIS 还完全陌生”的状态，我建议你按下面方式开展第一次体验：

1. 不要直接上 `research-pipeline`
2. 先准备一个简单但具体的研究题目
3. 在独立项目目录中写好 `CLAUDE.md`
4. 再写一个简短 `RESEARCH_BRIEF.md`
5. 先只跑 `/idea-discovery`
6. 仔细阅读输出文件
7. 手工确认 idea 后，再进入 `/experiment-bridge`
8. 第一次 `auto-review-loop` 务必开 `human checkpoint: true`
9. 写作阶段再进入 `/paper-writing`

这样你会真正“学会系统”，而不是只是“让系统跑过一遍”。

## 12. 结语

如果把这整套仓库压缩成一句话，ARIS 不是一个单一程序，而是一套以 skill 为核心、以项目文件为状态、以宿主 Agent 为执行器、以 reviewer 模型为外部批判者的自动科研操作系统。

对你来说，真正应该记住的核心点只有五个：

1. `Auto-claude-code-research-in-sleep` 是主线技能仓库，不是你的研究项目目录。
2. 真正使用时，要在独立研究项目目录中启动宿主 Agent。
3. `CLAUDE.md`、`RESEARCH_BRIEF.md`、`docs/research_contract.md`、`EXPERIMENT_LOG.md`、`AUTO_REVIEW.md` 是理解和恢复项目状态的关键文件。
4. skill 不是脚本，而是驱动 Agent 执行完整工作流的操作规范。
5. 对初学者来说，分步工作流比直接一条 `/research-pipeline` 更合适。

当你已经读完本文档后，你应该能够回答下面这些问题：

- `~/ARIS` 里每个 worktree / 分支是做什么的
- 主线仓库中每个主要目录承担什么角色
- ARIS 依赖哪些工具、模型、MCP 和可选服务
- skill 是怎么被宿主发现和调用的
- 执行模型和 reviewer 模型如何分工
- 一项科研从 idea discovery 到 experiment bridge 到 auto-review-loop 到 paper-writing 是怎么串起来的
- 每一步会生成哪些关键文件，下一步如何接着用

如果后面你愿意，我还可以继续基于这份手册，直接帮你在一个新的真实研究项目目录里完成：

- `CLAUDE.md` 初始化
- `RESEARCH_BRIEF.md` 起草
- 第一次 `/idea-discovery` 的运行准备
- 第一轮项目目录结构搭建

这样你就可以从“理解 ARIS”直接进入“开始第一次实际使用 ARIS”。
