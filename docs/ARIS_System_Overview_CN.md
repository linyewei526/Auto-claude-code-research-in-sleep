# ARIS 系统总说明（基于 `/data/home/wly/ARIS` 当前仓库状态）

## 1. 这份文档的定位

这份文档的目标不是重复 README，而是把你现在服务器上的 `/data/home/wly/ARIS` 目录整体拆开讲清楚，让你看完以后能回答下面这些问题：

- ARIS 到底是什么，默认运行形态是什么
- `/data/home/wly/ARIS` 里为什么同时有 `Auto-claude-code-research-in-sleep`、`aris-code`、`paper-poster`、`paper-slides`、`personal`
- 这些目录和 Git 分支是什么关系
- ARIS 的默认架构是什么，执行者和审稿者分别是谁
- 主线仓库为什么主要由 Markdown 技能文件组成
- 代码库每个分支、每个核心子目录、每类文件分别负责什么
- ARIS 运行时依赖哪些外部工具、哪些是必须、哪些是可选
- ARIS 从输入研究方向到产出实验、论文、rebuttal 的工作链路是怎样的
- 你未来应该在哪个目录里真正开展科研项目，而不是误把工具仓库本身当成研究项目目录

---

## 2. 先把你当前目录布局讲清楚

### 2.1 你的 `/data/home/wly/ARIS` 不是一个单独 Git 仓库

`/data/home/wly/ARIS` 顶层本身不是 Git 仓库。真正的 Git 仓库是：

- `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`

你现在看到的其他目录：

- `/data/home/wly/ARIS/aris-code`
- `/data/home/wly/ARIS/paper-poster`
- `/data/home/wly/ARIS/paper-slides`
- `/data/home/wly/ARIS/personal`

本质上都是你在 `Auto-claude-code-research-in-sleep` 这个仓库上，用 `git worktree` 拉出来的不同分支工作树。

### 2.2 当前 worktree 和分支的对应关系

根据仓库实际状态：

| 目录 | 对应分支 | 性质 | 作用 |
|---|---|---|---|
| `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep` | `main` | 主工作树 | ARIS 主线仓库，默认文档、技能、MCP bridge、模板、测试都在这里 |
| `/data/home/wly/ARIS/aris-code` | `aris-code` | 独立分支工作树 | ARIS 的独立 CLI 版本源码，Rust 工程 |
| `/data/home/wly/ARIS/paper-poster` | `feature/paper-poster` | 功能分支工作树 | `paper-poster` 能力的功能分支快照 |
| `/data/home/wly/ARIS/paper-slides` | `feature/paper-slides` | 功能分支工作树 | `paper-slides` 能力的功能分支快照 |
| `/data/home/wly/ARIS/personal` | `personal` | 你的个人分支工作树 | 你自己的记录、配置、说明文档区域 |

### 2.3 这些分支之间的真实关系

要点是：

- `main` 分支是主线分支，默认推荐使用它。
- `aris-code` 分支不是简单补丁，而是一套“独立 CLI 程序”的实现分支。
- `feature/paper-poster`、`feature/paper-slides` 是历史功能分支快照；其中对应能力在主线 README 中已经作为功能被纳入整个 ARIS 叙事，但你本地仍保留了单独 worktree。
- `personal` 不是上游功能分支，而是你自己的私人记录分支。

所以，**你今后真正拿来“按 README 用 ARIS”的主目录，是 `Auto-claude-code-research-in-sleep`，不是 `personal`，也不是 `aris-code`。**

---

## 3. ARIS 到底是什么

### 3.1 ARIS 不是传统意义上的“大型软件平台”

ARIS 在主线 `main` 分支中的核心思想是：

- 它首先是一套**科研工作流方法论**
- 然后是一套**以 Markdown 技能文件为载体的 Agent 工作流定义**
- 它依赖 Claude Code、Codex、Gemini、GLM、MiniMax 之类外部 Agent/LLM 执行这些技能
- 它本身并不是一个必须启动的后端服务，也不是必须部署的 Web 平台

主线 README 反复强调的一句话可以概括为：

- **ARIS 是 methodology，不是 platform。**

换句话说：

- ARIS 主线最重要的资产不是 Python 框架，而是 `skills/*/SKILL.md`
- 这些 `SKILL.md` 文件规定了“当用户说 `/idea-discovery` 或 `/research-pipeline` 时，Agent 应该如何分步骤思考、调用工具、生成文件、运行实验、接受外部评审”

### 3.2 ARIS 的默认架构是“执行者 + 外部审稿者”

主线默认设计是双角色：

1. **Executor，执行者**
   - 默认是 Claude Code
   - 负责读文件、写代码、查文献、生成实验脚本、修改论文、运行工作流

2. **Reviewer，审稿者**
   - 默认是通过 Codex MCP 接入的 GPT-5.4
   - 负责审稿、挑错、质疑、要求补实验、进行对抗式 review

这就是 README 里最核心的“跨模型协作”。

### 3.3 为什么要分执行者和审稿者

因为 ARIS 认为：

- 同一个模型又写又审，容易“自我宽容”
- 执行者和审稿者分开，才更接近真实研究中的“作者 vs reviewer”
- 最大提升不是从 2 个 reviewer 变成 4 个 reviewer，而是从“单模型自审”变成“跨模型对抗审查”

### 3.4 ARIS 的两种主要实现形态

你本地仓库里其实同时放着两种 ARIS：

#### 形态 A：主线 Markdown Skill 版

目录：

- `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`

特点：

- 以 `skills/*.md` 为核心
- 默认跑在 Claude Code 中
- 也可以适配 Codex CLI、Cursor、Trae、Antigravity 等
- 是仓库当前最完整、最成熟、文档最多的主线

#### 形态 B：独立 CLI 版 ARIS-Code

目录：

- `/data/home/wly/ARIS/aris-code`

特点：

- 是 Rust 写的独立终端程序
- 内置技能、内置工具、内置配置逻辑
- 不再完全依赖 Claude Code 的外壳
- 更像是“把 ARIS 的核心运行时做成了一个独立可执行程序”

对你当前服务器而言，**优先级应该是先掌握主线 Markdown Skill 版，再考虑是否研究 `aris-code`。**

---

## 4. ARIS 主线系统架构

下面讲的是 `main` 分支，也就是 `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`。

### 4.1 整体架构图

可以把主线 ARIS 理解成下面这条链：

1. 你在 Claude Code 里输入一个 slash 命令
   - 例如 `/idea-discovery`
   - 例如 `/research-pipeline`
   - 例如 `/paper-writing`

2. Claude Code 加载对应的 `SKILL.md`
   - `skills/idea-discovery/SKILL.md`
   - `skills/research-pipeline/SKILL.md`
   - `skills/paper-writing/SKILL.md`

3. 这个技能文件规定：
   - 允许调用哪些工具
   - 允许调用哪些子技能
   - 每个阶段该做什么
   - 该生成哪些中间文件
   - 在什么地方需要外部 reviewer
   - 什么时候需要 checkpoint、什么时候自动继续

4. Claude Code 按技能文件执行
   - 读项目文件
   - 调用 WebSearch/WebFetch
   - 读写 Markdown
   - 写实验脚本
   - 调用 `/run-experiment`
   - 调用 Codex MCP 做 review

5. 所有中间状态不依赖数据库，而是写回项目目录中的 Markdown/JSON 文件
   - `IDEA_REPORT.md`
   - `AUTO_REVIEW.md`
   - `REVIEW_STATE.json`
   - `refine-logs/EXPERIMENT_PLAN.md`
   - `docs/research_contract.md`
   - `paper/` 目录

6. 这些文件又成为后续技能的输入
   - 于是形成“磁盘文件驱动的多阶段科研流水线”

### 4.2 为什么说 ARIS 是“纯 Markdown 编排”

因为主线没有一个中心 Python orchestrator 去硬编码完整流程。

相反，流程逻辑主要散落在：

- `skills/*/SKILL.md`
- 项目约定文件
- Agent 自身的工具能力
- MCP bridge

这意味着：

- 优点：非常轻量，易 fork，易改写，换平台容易
- 缺点：理解门槛高，因为逻辑不是集中在一个 `main.py` 里，而是分散在几十个 skill 中

### 4.3 主线最重要的运行时约定

ARIS 主线真正的“运行时”不是进程，而是一组约定：

#### 约定 1：技能驱动

所有主流程都通过技能触发。

#### 约定 2：项目文件驱动

状态写在项目目录里，而不是写在数据库里。

#### 约定 3：`CLAUDE.md` 是项目仪表盘

项目根目录中的 `CLAUDE.md` 不是仓库文档，而是**你具体科研项目的运行说明书**。它负责告诉执行者：

- 当前阶段
- 当前 idea
- 服务器怎么连
- GPU 怎么用
- W&B 怎么配
- 下一步是什么

#### 约定 4：外部 reviewer 默认通过 MCP bridge 注入

主线默认 reviewer 是 Codex MCP，但也可以换成：

- `llm-chat`
- `minimax-chat`
- `gemini-review`
- `claude-review`

#### 约定 5：每个工作流都有对应产物

工作流的完成与否，不靠“模型说完成了”，而靠具体文件是否被生成、更新。

---

## 5. ARIS 主线工作流怎么串起来

### 5.1 工作流 1：Idea Discovery

核心技能：

- `research-lit`
- `idea-creator`
- `novelty-check`
- `research-review`
- `research-refine`
- `experiment-plan`
- `research-refine-pipeline`
- 总封装：`idea-discovery`

作用：

- 输入一个研究方向
- 搜文献
- 生成 8 到 12 个 idea
- 查新
- 外部 reviewer 批判
- 细化为更可实现的方法方案
- 产出实验计划

关键产物：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`
- 可选：`IDEA_CANDIDATES.md`

### 5.2 工作流 1.5：Experiment Bridge

核心技能：

- `experiment-bridge`
- `run-experiment`
- `monitor-experiment`
- `training-check`
- `result-to-claim`
- `ablation-planner`
- `vast-gpu`
- `serverless-modal`

作用：

- 把实验计划变成真实代码和真实实验
- 可先对代码做 GPT review
- 先跑 sanity，再跑 baseline/main/ablation
- 部署到本地 GPU、远程服务器、Vast.ai 或 Modal

关键产物：

- `refine-logs/EXPERIMENT_TRACKER.md`
- `EXPERIMENT_LOG.md`
- `findings.md`
- 结果 JSON/CSV/log

### 5.3 工作流 2：Auto Review Loop

核心技能：

- `auto-review-loop`

作用：

- 读取当前研究结果和论文叙事
- 交给外部 reviewer 打分
- 根据 reviewer 的 action items 自动修复
- 必要时触发更多实验
- 循环直到达到阈值或达到最大轮数

关键产物：

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- 可选：`REVIEWER_MEMORY.md`
- 可选：`CLAIMS_FROM_RESULTS.md`

### 5.4 工作流 3：Paper Writing

核心技能：

- `paper-plan`
- `paper-figure`
- `paper-write`
- `paper-compile`
- `auto-paper-improvement-loop`
- 总封装：`paper-writing`

作用：

- 从研究叙事或结果总结出发
- 规划论文结构
- 生成图表和表格
- 写 LaTeX
- 编译 PDF
- 再让 reviewer 批，自动润色两轮

关键产物：

- `PAPER_PLAN.md`
- `figures/`
- `paper/`
- `paper/main.pdf`
- `paper/PAPER_IMPROVEMENT_LOG.md`

### 5.5 工作流 4：Rebuttal

核心技能：

- `rebuttal`

作用：

- 处理外部真实审稿意见
- 解析 reviewer concern
- 逐条建立 issue board
- 根据 venue 限制起草 rebuttal
- 做安全检查：不编造、不越权承诺、全覆盖

关键产物：

- `rebuttal/ISSUE_BOARD.md`
- `rebuttal/STRATEGY_PLAN.md`
- `rebuttal/PASTE_READY.txt`
- `rebuttal/REBUTTAL_DRAFT_rich.md`

### 5.6 元工作流：Meta Optimize

核心技能：

- `meta-optimize`

作用：

- 通过 hooks 记录技能调用、失败、参数覆盖
- 反过来分析 SKILL.md 哪些地方应该改进

关键产物：

- `.aris/meta/events.jsonl`
- 技能优化建议文档

---

## 6. 主线仓库目录详解

下面全部以：

- `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`

为准。

### 6.1 顶层文件

| 路径 | 作用 |
|---|---|
| `.env.example` | 环境变量示例 |
| `.gitattributes` | Git 属性配置 |
| `.gitignore` | 忽略规则 |
| `LICENSE` | 许可证 |
| `CONTRIBUTING.md` / `CONTRIBUTING_CN.md` | 贡献说明 |
| `README.md` / `README_CN.md` | 主线总文档 |
| `xhs_post.md` | 项目宣传/分享类文案 |

### 6.2 `assets/`

作用：

- 存放 README 和文档用到的静态资源图
- 如社区案例截图、飞书通知截图、插图 demo 等

当前主要内容：

- `community_showcase_*.png`
- `feishu_*.png/jpg`
- `paper_illustration_demo.png`

### 6.3 `docs/`

作用：

- 存放扩展说明文档
- 不是核心运行逻辑，但承担“如何安装、如何替代 reviewer、如何适配其他 IDE/平台”的知识载体

当前关键文档分组如下：

#### 核心运行相关

| 文件 | 作用 |
|---|---|
| `PROJECT_FILES_GUIDE.md` / `PROJECT_FILES_GUIDE_CN.md` | 解释项目状态文件体系 |
| `SESSION_RECOVERY_GUIDE.md` / `SESSION_RECOVERY_GUIDE_CN.md` | 解释会话恢复、Pipeline Status、Hooks |
| `WATCHDOG_GUIDE.md` / `WATCHDOG_GUIDE_CN.md` | 解释远程 watchdog 持续监控 |
| `NARRATIVE_REPORT_EXAMPLE.md` | 工作流 3 输入示例 |

#### reviewer / 模型替代 / 后端替代

| 文件 | 作用 |
|---|---|
| `LLM_API_MIX_MATCH_GUIDE.md` | 任意执行器 + 任意 OpenAI 兼容 reviewer 的混搭指南 |
| `MINIMAX_MCP_GUIDE.md` | 用 MiniMax 替代 Codex reviewer |
| `MODELSCOPE_GUIDE.md` | 用 ModelScope 免费 API 跑完整流程 |
| `MiniMax-GLM-Configuration.md` | GLM + MiniMax 的组合方案 |
| `ALI_CODING_PLAN_GUIDE.md` | 阿里百炼 / Coding Plan 方案 |
| `CODEX_CLAUDE_REVIEW_GUIDE*.md` | Codex 执行 + Claude reviewer overlay |
| `CODEX_GEMINI_REVIEW_GUIDE*.md` | Codex 执行 + Gemini reviewer overlay |

#### IDE / 平台适配

| 文件 | 作用 |
|---|---|
| `CURSOR_ADAPTATION.md` | Cursor 适配 |
| `OPENCLAW_ADAPTATION.md` | OpenClaw 适配 |
| `TRAE_ARIS_RUNBOOK_*.md` | Trae 适配 |
| `ANTIGRAVITY_ADAPTATION*.md` | Antigravity 适配 |

#### 独立 CLI 转向说明

| 文件 | 作用 |
|---|---|
| `ARIS-Code-README_CN.md` / `ARIS-Code-README_EN.md` | 指向 `aris-code` 分支的说明入口 |

### 6.4 `templates/`

作用：

- 给你准备工作流输入模板
- 同时也定义 compact mode 使用的文件结构模板

关键模板：

| 文件 | 对应工作流 | 作用 |
|---|---|---|
| `RESEARCH_BRIEF_TEMPLATE.md` | Workflow 1 | 详细研究简报 |
| `RESEARCH_CONTRACT_TEMPLATE.md` | Workflow 1/恢复 | 当前 idea 的聚焦合同 |
| `EXPERIMENT_PLAN_TEMPLATE.md` | Workflow 1.5 | claim-driven 实验路线图 |
| `NARRATIVE_REPORT_TEMPLATE.md` | Workflow 3 | 论文叙事输入 |
| `PAPER_PLAN_TEMPLATE.md` | Workflow 3 | 论文大纲模板 |
| `IDEA_CANDIDATES_TEMPLATE.md` | compact | 存活 idea 候选池 |
| `EXPERIMENT_LOG_TEMPLATE.md` | compact | 实验日志 |
| `FINDINGS_TEMPLATE.md` | compact | 发现日志 |
| `claude-hooks/meta_logging.json` | meta-optimize | Claude Code hooks 示例配置 |

### 6.5 `tools/`

作用：

- 主线仓库里的辅助脚本
- 不直接承载整个工作流，但为技能补足联网抓取、bridge 支撑和监控能力

| 文件 | 作用 |
|---|---|
| `arxiv_fetch.py` | 调用 arXiv 获取论文信息/下载 |
| `semantic_scholar_fetch.py` | 调用 Semantic Scholar 获取结构化论文数据 |
| `convert_skills_to_llm_chat.py` | 将依赖 Codex reviewer 的 skill 改写为使用通用 `llm-chat` |
| `generate_codex_claude_review_overrides.py` | 生成/辅助 reviewer overlay 相关内容 |
| `watchdog.py` | 远程实验健康监控守护进程 |
| `meta_opt/check_ready.sh` | 元优化前检查日志数据是否足够 |
| `meta_opt/log_event.sh` | 元优化日志事件记录 |

### 6.6 `mcp-servers/`

作用：

- 提供 reviewer 或外部服务的 MCP bridge
- 这些 bridge 让 Claude Code / Codex 可以通过本地标准接口调用外部模型或服务

| 子目录 | 作用 |
|---|---|
| `claude-review/` | 给 Codex 路线提供 Claude reviewer bridge |
| `gemini-review/` | 给 Codex 路线提供 Gemini reviewer bridge，支持异步和本地图像审查 |
| `llm-chat/` | 通用 OpenAI-compatible Chat Completions MCP bridge |
| `minimax-chat/` | MiniMax 专用 reviewer MCP bridge |
| `feishu-bridge/` | 飞书双向交互桥服务 |

#### 各 bridge 的关系

- `claude-review`、`gemini-review` 是“review-only interface”，接口契约与 ARIS review 技能强绑定
- `llm-chat`、`minimax-chat` 更接近通用 Chat API bridge
- `feishu-bridge` 与 reviewer 无关，属于通知/审批交互层

### 6.7 `tests/`

作用：

- 测试本仓库中的 bridge 和 watchdog 等实际脚本

当前测试对象：

| 文件 | 作用 |
|---|---|
| `test_feishu_bridge_server.py` | 测飞书桥 |
| `test_llm_chat_server.py` | 测通用 LLM Chat MCP |
| `test_minimax_chat_server.py` | 测 MiniMax MCP |
| `test_minimax_integration.py` | 测 MiniMax 集成 |
| `test_watchdog.py` | 测远程 watchdog |

### 6.8 `skills/`

这是主线仓库最核心的目录。

#### 6.8.1 `skills/` 下的特殊目录

| 路径 | 作用 |
|---|---|
| `shared-references/` | 供 `paper-plan`、`paper-write` 等复用的写作规范与 venue checklist |
| `skills-codex/` | Codex CLI 原生技能包 |
| `skills-codex-claude-review/` | Codex + Claude reviewer overlay |
| `skills-codex-gemini-review/` | Codex + Gemini reviewer overlay |

#### 6.8.2 核心工作流技能一览

下面按功能分组列出 `skills/` 下主要子目录。

##### A. 文献、idea、查新、方案精炼

| 技能目录 | 作用 |
|---|---|
| `research-lit/` | 多源文献检索与综述 |
| `semantic-scholar/` | 补充 Semantic Scholar 搜索 |
| `arxiv/` | 直接查找/下载 arXiv 论文 |
| `idea-creator/` | 生成和排序研究 idea |
| `idea-discovery/` | 工作流 1 总封装 |
| `idea-discovery-robot/` | 面向机器人/具身领域的 idea 发现变体 |
| `novelty-check/` | 查新、对比最近文献、判断 novelty |
| `research-review/` | 外部 reviewer 深度评审研究方向/方案 |
| `research-refine/` | 把模糊方案打磨成 problem-anchored 计划 |
| `experiment-plan/` | 生成 claim-driven 实验路线图 |
| `research-refine-pipeline/` | `research-refine + experiment-plan` 串联封装 |

##### B. 实验实施与训练监控

| 技能目录 | 作用 |
|---|---|
| `experiment-bridge/` | 工作流 1.5 总封装，负责从计划到实验 |
| `run-experiment/` | 部署实验到本地/远程/Vast/Modal |
| `monitor-experiment/` | 监控实验、拉结果、汇总 |
| `training-check/` | 用 W&B 等信号检查训练质量 |
| `result-to-claim/` | 把实验结果压缩成可写入论文的 claim |
| `ablation-planner/` | 规划 ablation |
| `vast-gpu/` | Vast.ai GPU 生命周期管理 |
| `serverless-modal/` | Modal 无服务器 GPU |
| `system-profile/` | 性能剖析 |

##### C. 自动评审与科研迭代

| 技能目录 | 作用 |
|---|---|
| `auto-review-loop/` | 工作流 2，总自动 review 循环 |
| `auto-review-loop-llm/` | reviewer 切成通用 `llm-chat` 的版本 |
| `auto-review-loop-minimax/` | reviewer 切成 MiniMax 的版本 |
| `research-pipeline/` | 工作流 1 + 1.5 + 2 的总封装 |

##### D. 论文写作、图表、编译、海报、幻灯片

| 技能目录 | 作用 |
|---|---|
| `paper-plan/` | 论文结构规划 |
| `paper-figure/` | 自动图表与表格生成 |
| `paper-illustration/` | Gemini/图形相关插图生成 |
| `paper-write/` | LaTeX 分节写作 |
| `paper-compile/` | LaTeX 编译与修错 |
| `paper-writing/` | 工作流 3 总封装 |
| `auto-paper-improvement-loop/` | 论文写作层面的 reviewer 改进循环 |
| `paper-poster/` | 论文海报生成 |
| `paper-slides/` | 报告幻灯片生成 |
| `mermaid-diagram/` | Mermaid 图生成 |
| `pixel-art/` | 像素风图像类能力 |

##### E. 投稿后与学术写作扩展

| 技能目录 | 作用 |
|---|---|
| `rebuttal/` | 工作流 4，投稿后 rebuttal |
| `grant-proposal/` | 基金申请书生成 |
| `proof-writer/` | 理论证明写作 |
| `formula-derivation/` | 数学推导辅助 |

##### F. 行业/领域专用扩展

| 技能目录 | 作用 |
|---|---|
| `comm-lit-review/` | 通信/无线领域文献综述 |
| `dse-loop/` | 体系结构 / EDA 设计空间搜索 |

##### G. 通知、辅助、元优化

| 技能目录 | 作用 |
|---|---|
| `feishu-notify/` | 飞书通知与交互工具技能 |
| `meta-optimize/` | 通过使用日志反向优化技能 |
| `analyze-results/` | 分析实验结果 |

### 6.9 主线仓库中最重要的思想：技能才是主程序

看到这里应该能理解：

- 在主线分支中，`skills/` 才是“系统行为定义”
- `tools/`、`mcp-servers/`、`templates/`、`docs/` 都是在为这些技能服务

---

## 7. `aris-code` 分支详解

目录：

- `/data/home/wly/ARIS/aris-code`

这是另一套实现，不是单纯的 README 变体。

### 7.1 它是什么

它是 ARIS 的独立 CLI 版本：

- 不是“复制 `skills/` 到 `~/.claude/skills` 再让 Claude Code 读”
- 而是“用 Rust 写了一个自己的运行时、工具系统、配置系统、REPL、会话压缩、review 工具”

### 7.2 它的仓库顶层

| 路径 | 作用 |
|---|---|
| `Cargo.toml` | Rust workspace 顶层配置 |
| `Cargo.lock` | 依赖锁定 |
| `README*.md` | 独立 CLI 文档 |
| `CHANGELOG.md` | 版本日志 |
| `docs/screenshot.png` | CLI 截图 |

### 7.3 Rust workspace 的 crate 结构

`aris-code/crates` 下有 6 个 crate：

| crate | 作用 |
|---|---|
| `api` | LLM API 层，负责消息请求、SSE、类型封装 |
| `runtime` | 运行时核心，包含会话、工具、配置、MCP、权限、prompt、hooks、压缩 |
| `aris-cli` | CLI 入口、REPL、交互式 setup、会话控制 |
| `commands` | slash command 规格与帮助信息 |
| `tools` | 内置工具系统与 `LlmReview` 等工具定义 |
| `compat-harness` | 与上游/兼容层提取命令和工具清单的辅助库 |

### 7.4 `runtime` crate 的模块职责

根据 `crates/runtime/src` 实际文件：

| 文件 | 作用 |
|---|---|
| `bash.rs` | shell 命令执行 |
| `bootstrap.rs` | 启动引导阶段 |
| `compact.rs` | 会话压缩与续写 |
| `config.rs` | 配置加载与解析 |
| `conversation.rs` | 对话运行时与事件流 |
| `file_ops.rs` | 读写文件、grep、glob、edit |
| `hooks.rs` | hook 执行 |
| `mcp*.rs` | MCP 客户端与 stdio/remote transport |
| `oauth.rs` | OAuth 相关 |
| `permissions.rs` | 权限模式 |
| `prompt.rs` | 系统提示词构造 |
| `remote.rs` | 远程上下文、代理、token |
| `session.rs` | 会话消息结构 |
| `usage.rs` | token/成本统计 |

### 7.5 `aris-cli` crate 的角色

`aris-cli` 是用户直接运行的程序入口，负责：

- 读取保存的 ARIS 配置
- 首次无 API key 时触发 setup wizard
- 解析命令行参数
- 进入 REPL
- 支持 `/setup`、`/skills`、`/status`、`/permissions` 等 slash command

### 7.6 `tools` crate 的意义

这个 crate 说明 ARIS-Code 已经不是“完全依赖 Claude Code 提供工具”，而是在自己内部声明了一套工具规范，包括：

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

### 7.7 ARIS-Code 与主线 skill 版的关系

两者不是互斥关系，而是两条产品路线：

- 主线 skill 版：把 ARIS 当作一套可被 Claude Code/Codex 等加载的工作流
- ARIS-Code：把 ARIS 运行时自己实现出来，形成独立 CLI

### 7.8 对你当前服务器的现实意义

这一点很重要：

- `aris-code` README 明确强调当前 release 主打 **macOS Apple Silicon**
- 你的环境是 Linux 服务器
- 因此对你来说，**`aris-code` 更适合作为理解架构的参考，不适合作为你现在的首选落地路径**

也就是说，现阶段你应优先用主线 `main` 分支的 Markdown skill 版。

---

## 8. `paper-poster` 与 `paper-slides` worktree 的意义

这两个目录分别对应：

- `feature/paper-poster`
- `feature/paper-slides`

它们的意义主要是：

- 保留历史功能分支快照
- 让你可以单独检查 poster/slides 功能在功能分支上的实现状态

但对你当前使用 ARIS 来说：

- 这些不是主入口
- 你无需进入这些目录开展日常使用
- 只在你想比较功能分支与主线差异时才需要看

---

## 9. ARIS 运行所需工具与环境基础

下面按“必须”和“按功能需要”区分。

### 9.1 最小必需条件：主线技能版

如果你要按主线 README 使用 ARIS，最小需要：

1. **一个能加载 SKILL.md 的执行器环境**
   - 默认推荐：Claude Code
   - 可替代：Codex CLI、Cursor、Trae、Antigravity 等

2. **仓库技能文件**
   - 从 `skills/` 拷贝到对应工具的技能目录

3. **至少一个可用 LLM 后端**
   - 执行者本身总得能工作

### 9.2 如果你按默认主线组合跑

默认组合是：

- Executor：Claude Code
- Reviewer：Codex MCP 上的 GPT-5.4

因此需要：

1. Claude Code 可用
2. Codex CLI 可用
3. `claude mcp add codex -s user -- codex mcp-server`

### 9.3 如果你只做文献调研与方案设计

需要：

- 执行器
- 网络
- 可选 reviewer

不一定需要：

- GPU
- 远程服务器
- LaTeX

### 9.4 如果你要做实验桥接

还需要：

- Python 运行环境
- 你的项目代码能运行
- 本地 GPU、远程 GPU、Vast.ai 或 Modal 至少一种
- SSH、`screen`/`tmux` 等远程执行基础设施

### 9.5 如果你要做论文写作

还需要：

- LaTeX
- `latexmk`
- `pdfinfo`
- 通常还需要 `pdftotext`

### 9.6 如果你要用可选增强功能

| 功能 | 需要 |
|---|---|
| Zotero 集成 | Zotero MCP 或 Web API |
| Obsidian 集成 | `mcpvault` 等 Obsidian MCP |
| W&B 监控 | `wandb` 登录和项目配置 |
| 飞书通知 | `~/.claude/feishu.json` 及相关 webhook/桥服务 |
| Vast.ai | `vastai` CLI + API Key + SSH key |
| Modal | `modal` CLI + 登录 |
| meta-optimize | Claude hooks + `.aris/meta/events.jsonl` |

---

## 10. ARIS 的项目文件体系

这是理解 ARIS 的关键，因为主线运行状态都靠这些文件维持。

### 10.1 推荐的项目级文件结构

根据 `PROJECT_FILES_GUIDE_CN.md`，一个真实研究项目目录最终会长这样：

```text
project/
├── CLAUDE.md
├── IDEA_CANDIDATES.md
├── findings.md
├── EXPERIMENT_LOG.md
├── docs/
│   └── research_contract.md
├── refine-logs/
│   ├── EXPERIMENT_PLAN.md
│   └── EXPERIMENT_TRACKER.md
├── AUTO_REVIEW.md
├── REVIEW_STATE.json
└── IDEA_REPORT.md
```

### 10.2 最关键的是 `CLAUDE.md`

在 ARIS 里，项目根目录的 `CLAUDE.md` 是：

- 当前阶段仪表盘
- 服务器配置文件
- 长任务恢复入口
- 研究约束说明

最重要的段落是：

- `## Pipeline Status`

### 10.3 为什么要有 `docs/research_contract.md`

因为 `IDEA_REPORT.md` 会很长，可能包含 8 到 12 个 idea。

进入真正实现阶段以后：

- 不应该每次都让 Agent 重新加载全部候选 idea
- 应该把当前选中的那个 idea 单独抽成 `docs/research_contract.md`

这就是 ARIS 的“上下文压缩友好”设计。

### 10.4 Compact 模式是什么意思

`compact: true` 的核心是：

- 不再频繁读取超长原始文件
- 而是维护更短、更稳定的摘要文件

例如：

- `IDEA_CANDIDATES.md`
- `EXPERIMENT_LOG.md`
- `findings.md`

这对长时间项目、会话恢复、上下文压缩非常重要。

---

## 11. ARIS 是怎么运行起来的

下面用一条真实链路说明。

### 11.1 你输入命令

例如：

```text
/research-pipeline "离散扩散语言模型的 factorized gap"
```

### 11.2 Agent 读取 `skills/research-pipeline/SKILL.md`

它会知道：

- 要先调用 `/idea-discovery`
- 然后进入实现阶段
- 然后 `/run-experiment`
- 然后 `/auto-review-loop`

### 11.3 `idea-discovery` 又继续拆子任务

它再调用：

- `/research-lit`
- `/idea-creator`
- `/novelty-check`
- `/research-review`
- `/research-refine-pipeline`

### 11.4 每一步都把结果写回文件

例如：

- 文献综述写到报告
- idea 写到 `IDEA_REPORT.md`
- refine 写到 `refine-logs/FINAL_PROPOSAL.md`
- 实验计划写到 `refine-logs/EXPERIMENT_PLAN.md`

### 11.5 reviewer 的介入方式

当技能要求外部审稿时：

- 默认通过 `mcp__codex__codex`
- 或通过其它 MCP bridge

reviewer 输出：

- 分数
- weakness
- minimum fix
- 是否 ready

### 11.6 实验执行方式

如果需要实验：

- `run-experiment` 先读项目 `CLAUDE.md`
- 根据其中的 `gpu: local / remote / vast / modal`
- 决定部署到哪里
- 再由 `monitor-experiment` 拉结果

### 11.7 自动 review loop 如何结束

当出现以下情况之一时结束：

- 分数达到阈值
- verdict 达到 ready / almost
- 轮数达到上限
- 人工 checkpoint 要求停止

### 11.8 论文写作如何承接前面的工作

通常顺序是：

1. 工作流 2 结束后，你已经有了结果与叙事
2. 你整理成 `NARRATIVE_REPORT.md`
3. `/paper-writing` 把它变成 `paper/` 和 PDF

---

## 12. 你真正应该在哪儿做科研项目

这一点很关键。

**不要把真正的科研项目直接做在 ARIS 工具仓库里。**

原因：

- ARIS 仓库是工具与技能仓库
- 你的研究项目会产生大量代码、实验、日志、论文文件
- 把两者混在一个 Git 仓库里会极其混乱

推荐做法：

1. 保留 `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep` 作为工具仓库
2. 另建一个独立科研项目目录
   - 例如 `/data/home/wly/research/my_first_aris_project`
3. 在那个项目目录里创建：
   - `CLAUDE.md`
   - `RESEARCH_BRIEF.md`
   - `papers/`
   - `docs/`
   - `refine-logs/`
4. 在那个项目目录中启动 Claude Code / Codex 并运行 ARIS 技能

---

## 13. 你现在应该如何理解“主线 ARIS”和“ARIS-Code”

### 13.1 对你当前目标来说，推荐优先级

推荐顺序是：

1. **先学会主线 skill 版**
2. **先在一个独立研究项目目录里跑通最小流程**
3. **有余力再研究 `aris-code` 独立 CLI**

### 13.2 为什么不是先用 `aris-code`

因为：

- 你当前目标是“把这个系统用起来”
- 主线 skill 版文档最丰富，生态最完整
- `aris-code` 当前在 README 中强调的是独立 CLI 路线，且发布说明偏向 macOS Apple Silicon
- 你的服务器环境更适合先按主线技能版理解整套方法论

---

## 14. 一句话总结

把 `/data/home/wly/ARIS` 这套东西看成：

- 一个主仓库 `Auto-claude-code-research-in-sleep`，里面装着 ARIS 主线技能系统
- 三个辅助/分支工作树：`aris-code`、`paper-poster`、`paper-slides`
- 一个你的私人记录工作树：`personal`

而把 ARIS 本身看成：

- 一套“执行者 + 外部审稿者”的自动科研工作流
- 以 `skills/*.md` 为核心编排逻辑
- 以项目目录里的 Markdown/JSON 文件作为状态存储
- 通过 MCP bridge 连接 reviewer、消息通知、替代模型和外部工具

你后续真正的重点，不是修改 ARIS 工具仓库本身，而是：

- 配好执行器
- 配好 reviewer
- 新建一个研究项目目录
- 在那个项目目录里写好 `CLAUDE.md`
- 再开始跑 `/idea-discovery`、`/research-pipeline`、`/paper-writing` 等工作流

