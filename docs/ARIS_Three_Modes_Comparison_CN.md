# ARIS 三种运行方式对比与工作流说明

## 1. 文档目的

这份文档专门回答你现在最关心的问题：

1. ARIS 在你当前仓库里，实际上支持哪几种主流使用方式。
2. 这三种方式分别是谁负责执行、谁负责审稿、技能装到哪里、工具怎么调用。
3. 这三种方式在当前代码库里的真实支持程度有什么差异。
4. 如果你的目标是“从找 idea 到写 code 到写作到 review 再修改”的整条链路，三种方式分别会怎么跑。

这里讨论的三种方式是：

1. 原生主线：Claude Code 科研与写作，Codex 做 reviewer
2. Codex 主执行：Codex 科研与写作，Claude 做 reviewer
3. 纯 Codex：Codex 既做科研与写作，也做 reviewer，不依赖 Claude 或其他外部模型

---

## 2. 先给出总表

| 方式 | 执行者 | 审稿者 | skills 安装位置 | reviewer 调用方式 | 当前仓库支持度 | 当前你机器上的可行性 |
|---|---|---|---|---|---|---|
| 方式 1 | Claude Code | Codex | `~/.claude/skills/` | Claude 通过 `codex` MCP 调 reviewer | 最高，主线默认路线 | 目前受你当前 Claude 服务不可用影响，暂时阻塞 |
| 方式 2 | Codex CLI | Claude Code | `~/.codex/skills/` | Codex 通过 `claude-review` MCP 调 reviewer | 有官方 overlay，但不是最完整主线 | 同样受当前 Claude 服务不可用影响，暂时阻塞 |
| 方式 3 | Codex CLI | 第二个 Codex reviewer agent | `~/.codex/skills/` | `spawn_agent` / `send_input` | 很强，且比方式 2 更新更完整 | 现在最可行，立即可尝试 |

---

## 3. 三种方式共用的 ARIS 主流程

无论你最后选哪条路线，ARIS 要做的科研流水线本质上都类似：

1. 文献调研：`/research-lit`
2. idea 生成与初筛：`/idea-creator`
3. 查新与批判：`/novelty-check`、`/research-review`
4. 方法精炼与实验规划：`/research-refine-pipeline`
5. 实验实现与部署：`/experiment-bridge`、`/run-experiment`、`/monitor-experiment`
6. 审稿驱动的改进循环：`/auto-review-loop`
7. 论文规划与写作：`/paper-plan`、`/paper-figure`、`/paper-write`、`/paper-compile`
8. 论文改进循环：`/auto-paper-improvement-loop`
9. 一键全流程封装：`/research-pipeline`

仓库主 README 里把这条主线直接写成：

- `idea-discovery -> experiment-bridge -> auto-review-loop -> paper-writing`

对应文件产物主要是：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `AUTO_REVIEW.md`
- `paper/`

项目状态文件规范见：

- [PROJECT_FILES_GUIDE_CN.md](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/docs/PROJECT_FILES_GUIDE_CN.md)

---

## 4. 方式 1：Claude Code 科研与写作，Codex 做 reviewer

### 4.1 这是仓库默认主线

这是 ARIS 的原生默认路线，也是 README 快速开始里的官方路径：

- skills 装到 `~/.claude/skills/`
- review-heavy skill 通过 Claude 里的 `codex` MCP 去调用 Codex reviewer

主 README 快速开始明确写了：

1. 复制 `skills/*` 到 `~/.claude/skills/`
2. 安装 Codex CLI
3. 运行 `claude mcp add codex -s user -- codex mcp-server`
4. 在 Claude Code 里直接使用 `/idea-discovery`、`/research-pipeline` 等命令

参考：

- [README_CN.md:125](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/README_CN.md#L125)

### 4.2 这条路线的工作分工

- Claude Code 负责执行：
  - 读仓库与项目文件
  - 写代码
  - 改实验脚本
  - 启动训练
  - 收集日志
  - 写论文草稿
  - 改 LaTeX
- Codex 负责 reviewer：
  - 打分
  - 找薄弱点
  - 提最低修复要求
  - 逼执行者补实验、改叙事、改图表、改论文

主 README 对这条分工的原话核心意思是：

- Claude Code 执行更快更顺
- Codex 审稿更严更深
- “速度 × 严谨”的互补强于单模型自审

参考：

- [README_CN.md:19](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/README_CN.md#L19)
- [README_CN.md:21](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/README_CN.md#L21)
- [README_CN.md:27](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/README_CN.md#L27)

### 4.3 这条路线里 skills 和工具怎么调用

#### skills 安装层

- 安装目录：`~/.claude/skills/`
- 安装内容：主线 `skills/*`

#### reviewer 调用层

- reviewer-heavy skills 在 Claude Code 中通过 `codex` MCP 调用 Codex
- 这意味着 Claude 是主 agent，Codex 是外部 reviewer 工具

#### 常见工作流调用链

1. `/idea-discovery`
   - Claude 先跑文献、整理候选方向
   - 需要 reviewer 的阶段调用 Codex
   - 产出 `IDEA_REPORT.md`
2. `/experiment-bridge`
   - Claude 根据计划写代码、部署实验、拉日志
3. `/auto-review-loop`
   - Codex 审稿
   - Claude 执行修复
   - 循环到达阈值或轮次上限
4. `/paper-writing`
   - Claude 规划、作图、写论文、编译 PDF
   - reviewer 阶段仍由 Codex 来挑问题

### 4.4 优点

- 文档最完整
- 仓库维护优先级最高
- 与 README、模板、主线叙事完全一致
- reviewer 和 executor 明确分离

### 4.5 缺点

- 需要 Claude Code 真正可用
- 当前你的 Claude 中转服务虽然配置通了，但实际模型通道不可用，所以这条路线目前不能稳定落地

### 4.6 当前代码库下的实际评价

这是当前仓库里“最标准、最完整、最接近作者预期”的路线。

如果你未来把 Claude 服务恢复正常，它仍然应该是 ARIS 效果最稳的一条路。

---

## 5. 方式 2：Codex 科研与写作，Claude 做 reviewer

### 5.1 这是官方支持的 Codex-first 叠加路径

仓库专门给了这条路：

- `skills/skills-codex/` 作为基础技能包
- `skills/skills-codex-claude-review/` 作为 reviewer overlay
- `mcp-servers/claude-review/` 作为本地 bridge

参考：

- [CODEX_CLAUDE_REVIEW_GUIDE_CN.md](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/docs/CODEX_CLAUDE_REVIEW_GUIDE_CN.md)
- [skills-codex-claude-review/README_CN.md:3](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/README_CN.md#L3)

### 5.2 这条路线的工作分工

- Codex 是主执行者：
  - 读文件
  - 查文献
  - 写实验代码
  - 写论文和 LaTeX
  - 跑工作流
- Claude Code 是 reviewer：
  - 通过本地 `claude-review` MCP bridge 被 Codex 调用
  - 专门返回 reviewer 文字结果

### 5.3 skills 和工具调用方式

#### skills 安装层

安装顺序必须是：

1. 先装 `skills/skills-codex/*` 到 `~/.codex/skills/`
2. 再装 `skills/skills-codex-claude-review/*` 到 `~/.codex/skills/`

overlay 文档明确写了这一步：

- [skills-codex-claude-review/README_CN.md:28](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/README_CN.md#L28)

#### reviewer bridge 层

再注册本地 reviewer bridge：

- `codex mcp add claude-review -- python3 ~/.codex/mcp-servers/claude-review/server.py`

bridge 暴露的接口有两类：

- 同步：
  - `review`
  - `review_reply`
- 异步：
  - `review_start`
  - `review_reply_start`
  - `review_status`

参考：

- [mcp-servers/claude-review/README.md:5](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/claude-review/README.md#L5)
- [mcp-servers/claude-review/README.md:53](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/claude-review/README.md#L53)

#### 运行时 reviewer 调用

当前 overlay 的核心逻辑是：

- 把原来 `spawn_agent` / `send_input` 的第二个 Codex reviewer
- 改写成 `mcp__claude-review__review_start` / `review_reply_start` / `review_status`

这个改写是由仓库里的生成脚本完成的：

- [generate_codex_claude_review_overrides.py:16](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/tools/generate_codex_claude_review_overrides.py#L16)

### 5.4 这条路线名义上的完整工作流

1. 启动 Codex CLI
2. 在项目里调用：
   - `/idea-discovery`
   - `/experiment-bridge`
   - `/auto-review-loop`
   - `/paper-writing`
3. 其中 reviewer-heavy 环节由 Claude 做外部审稿

### 5.5 但当前代码库里的“实际落地情况”必须说清楚

这条路线不是坏的，但它目前有两个现实限制。

#### 限制 1：overlay 只覆盖 8 个核心 reviewer-heavy skill

官方 README 和 overlay 说明都承认它是“薄覆盖层”，只覆盖 8 个技能：

- `research-review`
- `novelty-check`
- `research-refine`
- `auto-review-loop`
- `paper-plan`
- `paper-figure`
- `paper-write`
- `auto-paper-improvement-loop`

参考：

- [skills-codex-claude-review/README_CN.md:13](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/README_CN.md#L13)
- [skills-codex-claude-review/README_CN.md:17](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/README_CN.md#L17)

这意味着：

- 它不是把整个 `skills-codex` 全量 reviewer 路径都切成 Claude
- 某些上层 wrapper 或中间技能仍然会沿用原来的 Codex native reviewer 逻辑

最典型的例子是：

- `idea-discovery` 会调用 `idea-creator`
- 但当前 `idea-creator` 仍然是 `spawn_agent` 第二个 Codex reviewer

参考：

- [idea-discovery/SKILL.md:12](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/idea-discovery/SKILL.md#L12)
- [idea-creator/SKILL.md:20](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/idea-creator/SKILL.md#L20)

所以，当前仓库里的方式 2，更准确的说法是：

- **Codex 主执行**
- **Claude 负责部分关键 review-heavy 环节的 reviewer**
- **不是所有 reviewer 都已经切换为 Claude**

#### 限制 2：Claude-review overlay 当前落后于 `skills-codex` 主版本

我核对过几个关键 skill，发现 `skills-codex-claude-review/*` 目前存在版本滞后。

例如：

- `auto-review-loop` 的 Claude overlay 少了 `COMPACT`、`training-check`、`findings.md`、`result-to-claim` 等更新能力
- `paper-write` 的 Claude overlay 只保留了 `ICLR/NeurIPS/ICML`，丢掉了主版里更完整的 `CVPR/ACL/AAAI/ACM/IEEE` 与 shared references 支持

这意味着：

- 方式 2 不是不能用
- 但它在当前仓库里不是最完整、最新的 reviewer 路径

### 5.6 优点

- 用 Codex 做主执行，能节省 Claude 配额
- 仍保留跨模型 reviewer 的核心思想
- 对长 review prompt 使用异步接口，设计上是合理的

### 5.7 缺点

- 依赖 Claude 服务可用
- 当前 overlay 不是全覆盖
- 当前 overlay 相比 `skills-codex` 主版本存在滞后

### 5.8 当前代码库下的实际评价

这条路线是“官方支持，但次于主线”的路径。

如果 Claude 账号额度紧张，它可以作为节流方案；
但如果你追求当前仓库里最完整、最稳的执行效果，它不如方式 1，也不如最新的纯 Codex 路径那么整齐。

---

## 6. 方式 3：Codex 科研与写作，Codex 也做 reviewer

### 6.1 这是 `skills-codex/*` 的原生形态

纯 Codex 路线不需要 Claude，不需要 Gemini，也不需要额外 reviewer bridge。

它直接使用：

- `skills/skills-codex/*`
- Codex 自己的主 agent 做执行
- 需要 reviewer 时再 `spawn_agent` 出第二个 Codex reviewer

### 6.2 这条路线的工作分工

- 主 Codex agent：
  - 执行科研和写作
- 第二个 Codex agent：
  - 扮演 reviewer

也就是说，这是一种“同平台双 agent”的 reviewer 结构，不是单 agent 自言自语。

### 6.3 skills 和工具调用方式

#### skills 安装层

- 安装目录：`~/.codex/skills/`
- 安装内容：只装 `skills/skills-codex/*`

#### reviewer 调用层

reviewer-aware skill 直接使用：

- `spawn_agent`
- `send_input`

例如 `research-review` 明确写着：

- reviewer model 默认 `gpt-5.4`
- 用第二个 Codex agent 执行审稿

参考：

- [research-review/SKILL.md:12](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/research-review/SKILL.md#L12)
- [research-review/SKILL.md:31](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/research-review/SKILL.md#L31)

`auto-review-loop`、`paper-write`、`paper-writing` 等也是同样的 reviewer 思路：

- [auto-review-loop/SKILL.md:17](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/auto-review-loop/SKILL.md#L17)
- [paper-writing/SKILL.md:25](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/paper-writing/SKILL.md#L25)
- [paper-write/SKILL.md:12](/data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/paper-write/SKILL.md#L12)

### 6.4 这条路线的完整工作流

1. 用 Codex 启动项目
2. 在项目里运行：
   - `/idea-discovery`
   - `/experiment-bridge`
   - `/auto-review-loop`
   - `/paper-writing`
3. 需要 reviewer 时，主 Codex agent 会调起第二个 reviewer Codex agent
4. 所有能力都留在同一个 Codex 生态里

### 6.5 优点

- 不依赖 Claude
- 不依赖额外 reviewer API
- 安装最简单
- 对你当前服务器条件来说，马上就能落地
- 当前 `skills-codex/*` 其实比 `skills-codex-claude-review/*` 更新更完整

### 6.6 缺点

- reviewer 和 executor 同属 Codex 体系，跨模型对抗性弱于方式 1 和方式 2
- 更容易出现“同风格偏差”

### 6.7 当前代码库下的实际评价

如果只看“现在就能在你机器上跑通”的现实性，这条路是最强的。

如果只看“跨模型 reviewer 的纯度”，它又弱于方式 1 和方式 2。

所以它的定位不是“理论最优”，而是“当前最可执行、最稳落地”。

---

## 7. 三种方式的逐项比较

### 7.1 从代码库成熟度看

排序：

1. 方式 1
2. 方式 3
3. 方式 2

原因：

- 方式 1 是主线默认
- 方式 3 使用的是当前更完整的 `skills-codex/*`
- 方式 2 虽有官方 overlay，但当前覆盖范围和版本同步都不如方式 3

### 7.2 从安装复杂度看

排序：

1. 方式 3 最简单
2. 方式 1 中等
3. 方式 2 最复杂

原因：

- 方式 3 不需要 reviewer bridge
- 方式 1 需要给 Claude 配 Codex MCP
- 方式 2 需要给 Codex 配 Claude-review bridge，并处理 overlay

### 7.3 从跨模型对抗性看

排序：

1. 方式 1
2. 方式 2
3. 方式 3

原因：

- 方式 1、2 都是跨模型执行者 vs reviewer
- 方式 3 是同平台双 agent reviewer

### 7.4 从你当前机器的现实可用性看

排序：

1. 方式 3
2. 方式 2
3. 方式 1

但这个排序不是因为方式 1 理论差，而是因为：

- 你当前 `~/.codex/config.toml` 已经可用
- `~/.codex` 当前没有 MCP，但很容易补
- 你当前的 Claude 中转服务虽然认证通了，但模型通道不可用

所以：

- 方式 1 当前不能直接实跑
- 方式 2 当前也不能稳定实跑
- 方式 3 现在就能开跑

---

## 8. 对你当前情况的直接建议

如果你的目标是：

- 先尽快把 ARIS 全流程真正跑一遍
- 先获得一次从 idea 到 code 到 paper 到 review 的完整体验

建议优先顺序是：

1. 先跑方式 3，纯 Codex
2. 等 Claude 服务恢复后，再尝试方式 1
3. 如果你仍然想压缩 Claude 配额，再考虑方式 2

原因很简单：

- 方式 3 现在立刻可行
- 方式 1 长期效果更优
- 方式 2 当前代码库里更像“可用但需要你接受一些 overlay 不完整现实”的折中方案

---

## 9. 最终结论

如果只问一句话：

- **ARIS 当前最标准的路线是方式 1。**
- **ARIS 当前在你机器上最现实、最容易立刻跑通的路线是方式 3。**
- **方式 2 是官方支持的折中路线，但当前仓库里并不是最完整、最推荐优先的新手起步路径。**

下一份文档会在这个判断基础上，专门按你当前服务器状态，给出三条路线各自的逐步操作指令和完整体验流程。
