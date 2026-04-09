# ARIS 三种路线在你当前服务器上的全流程实操指南

## 1. 这份文档解决什么问题

这份文档不再讨论抽象架构，而是直接回答：

- 在你现在这台服务器上，三种 ARIS 路线分别应该怎么装、怎么配、怎么跑
- 哪些路线现在能直接试
- 哪些路线当前会被你的 Claude 服务状态卡住
- 如果你想体验一次“找 idea -> 写 code -> 跑实验 -> 写论文 -> reviewer -> 修改”的全流程，具体命令怎么敲

本文件默认你已经完成：

- fork / clone / worktree
- `personal/docs/ARIS_Setting.md` 里写的 Git 准备

---

## 2. 你当前服务器的真实状态

以下信息是我在你当前服务器上直接检查得到的。

### 2.1 系统与工具版本

- 系统：Ubuntu 24.04.3 LTS
- 内核：Linux 6.17.0-19-generic
- Python：3.10.20
- Node.js：18.19.0
- npm：9.2.0
- git：2.43.0
- Claude Code：2.1.92
- Codex CLI：0.117.0

### 2.2 Codex 当前配置

你当前 `~/.codex/config.toml` 已经可用，关键配置是：

- `model_provider = "sub2api"`
- `model = "gpt-5.4"`
- `model_reasoning_effort = "high"`
- `/data/home/wly/ARIS` 已经被标记为 `trusted`

这意味着：

- 纯 Codex 路线现在具备立即运行的基础

### 2.3 Claude 当前配置状态

你当前：

- `~/.bashrc` 已设置 `ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL`
- `~/.claude/settings.json` 已被你之前停用

更关键的是：

- 你当前这个 Claude 中转服务虽然端点和鉴权链路是通的
- 但实测 `claude-opus-4-6` / `claude-opus-4-6-thinking` 返回的是 `No available channel`

这意味着：

- 依赖 Claude 真正返回结果的路线，现在都暂时不可稳定使用
- 所以方式 1 和方式 2 当前都属于“理论步骤已知，但前置服务阻塞”

### 2.4 GPU 资源状态

你这台机器是 4 张 A100 80GB。

我检查时的资源状态大致是：

- GPU 0：已占用约 31 GB
- GPU 1：已占用约 37 GB
- GPU 2：几乎空闲，仅几十 MB
- GPU 3：已占用约 31 GB

所以如果你现在要做第一次体验，最现实的选择是：

- 优先把本次实验绑定到 `GPU 2`

---

## 3. 先给结论：现在三条路线谁能直接跑

| 路线 | 现在是否可直接尝试 | 原因 |
|---|---|---|
| 方式 1：Claude 执行 + Codex reviewer | 不建议现在直接跑 | Claude 服务通道当前不可用 |
| 方式 2：Codex 执行 + Claude reviewer | 不建议现在直接跑 | reviewer 仍依赖 Claude 服务 |
| 方式 3：纯 Codex | 可以立即尝试 | 不依赖 Claude，Codex 已配置好 |

所以从“现在就想体验一次完整流程”这个目标出发：

- **先跑方式 3**
- **把方式 1 和方式 2 的步骤先准备好，等 Claude 服务恢复后再切换**

---

## 4. 三条路线共同的准备工作

不管你最后选哪条路径，都建议先准备一个独立项目目录，不要直接在 ARIS 工具仓库里做科研。

### Step 1：创建独立 demo 项目目录

```bash
mkdir -p /data/home/wly/research
mkdir -p /data/home/wly/research/aris_demo_project
cd /data/home/wly/research/aris_demo_project
```

### Step 2：复制几个基础模板

```bash
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_BRIEF_TEMPLATE.md ./RESEARCH_BRIEF.md
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/NARRATIVE_REPORT_TEMPLATE.md ./NARRATIVE_REPORT.md
mkdir -p docs refine-logs figures paper
```

### Step 3：同时准备 `CLAUDE.md` 和 `AGENTS.md`

这是一个很实际的兼容性处理。

原因：

- 主线 Claude 路线更强调 `CLAUDE.md`
- 但当前 `skills-codex/*` 中部分关键技能仍然读 `AGENTS.md`
- 尤其是 `run-experiment`、`research-lit`、`experiment-bridge`

所以最稳妥的做法是：

- 项目根目录里同时放 `CLAUDE.md` 和 `AGENTS.md`
- 两个文件内容保持一致

你可以直接创建如下内容。

先建 `CLAUDE.md`：

```bash
cat > /data/home/wly/research/aris_demo_project/CLAUDE.md <<'EOF'
# Project Dashboard

## Pipeline Status
- stage: idea-discovery
- goal: first end-to-end ARIS demo on this server
- current focus: run a lightweight full pipeline once

## Research Direction
- topic: improve a small ML baseline through ARIS full pipeline
- requirement: prefer lightweight experiments first, then expand if promising

## Compute
- gpu: local
- preferred_gpu: 2
- launch_prefix: CUDA_VISIBLE_DEVICES=2
- max_parallel_runs: 1

## Environment
- python: python3
- node: node
- repo_root: /data/home/wly/research/aris_demo_project
- code_sync: local
- wandb: false

## Constraints
- Do not use GPUs 0,1,3 unless I explicitly approve.
- Prefer short sanity experiments before any long run.
- Save all intermediate reports to project root or refine-logs/.
EOF
```

再复制成 `AGENTS.md`：

```bash
cp /data/home/wly/research/aris_demo_project/CLAUDE.md /data/home/wly/research/aris_demo_project/AGENTS.md
```

### Step 4：把 `RESEARCH_BRIEF.md` 改成你的真实方向

第一次体验时，不建议写得太空。

建议你直接写：

- 一个具体方向
- 一个参考论文
- 一个基础代码仓库

因为：

- 只给“模糊方向”，ARIS 也能跑
- 但如果你真的想体验“写 code + 跑实验 + 写论文”，最好给它一个 `ref paper` 和 `base repo`

你可以先写一个 demo 版，例如：

```markdown
# Research Brief

## Goal
Run one end-to-end ARIS demo on a manageable ML topic.

## Preferred Style
- Start with literature survey
- Generate multiple ideas
- Keep only ideas with realistic experiment cost
- Prefer one reproducible baseline repository

## Constraints
- Use local GPU 2 first
- Prefer experiments that can finish as a sanity run within 10-30 minutes
- Avoid giant multi-day training in the first demo

## Optional Inputs
- Reference paper: <在这里填论文 URL 或 PDF 路径>
- Base repo: <在这里填 GitHub repo URL>
```

---

## 5. 方式 1：Claude 执行 + Codex reviewer

## 5.1 这条路线当前的现实状态

当前不建议你现在就跑这条路线，原因不是步骤不清楚，而是：

- 你的 Claude 服务当前没有可用模型通道

所以这一节你应理解成：

- **这是恢复后该怎么做的标准操作手册**

### 5.2 安装步骤

#### Step 1：安装主线 skills 到 `~/.claude/skills/`

```bash
mkdir -p /data/home/wly/.claude/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/* /data/home/wly/.claude/skills/
```

#### Step 2：确保 Codex CLI 就绪

你已经有 Codex CLI 了，但还是建议确认：

```bash
codex --version
sed -n '1,80p' /data/home/wly/.codex/config.toml
```

#### Step 3：把 Codex 注册给 Claude 当 MCP

```bash
claude mcp add codex -s user -- codex mcp-server
```

验证：

```bash
claude mcp list
```

如果这一步成功，它通常会自动生成或更新 Claude 用户侧配置，不要求你手工重建旧的 `settings.json`。

#### Step 4：验证 Claude 本身可用

在你当前环境里，这一步大概率还会失败，除非服务商恢复通道：

```bash
claude -p "Reply with exactly READY" --output-format json --tools ""
```

只要这一步不通，整条方式 1 就不要继续往下跑。

### 5.3 运行全流程的方法

有两种跑法。

#### 跑法 A：分阶段学习，最适合第一次体验

进入项目：

```bash
cd /data/home/wly/research/aris_demo_project
claude
```

然后在 Claude 会话里按顺序跑：

```text
/idea-discovery "你的具体研究方向" — AUTO_PROCEED: false, compact: true
```

等它产出 `IDEA_REPORT.md`、`refine-logs/FINAL_PROPOSAL.md`、`refine-logs/EXPERIMENT_PLAN.md` 后，继续：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
```

再进入审稿循环：

```text
/auto-review-loop "你的主题" — human checkpoint: true, compact: true
```

等产出 `AUTO_REVIEW.md` 后，进入写作：

```text
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

#### 跑法 B：一条命令端到端

```text
/research-pipeline "你的研究方向" — AUTO_PROCEED: false, human checkpoint: true, compact: true
```

第一次不建议直接这么跑，因为：

- 你还不熟悉各阶段输出
- 中间更难发现问题到底卡在哪个 skill

### 5.4 这条路线的 reviewer 是怎么被调用的

在这条路线里：

- Claude 是主执行器
- reviewer-heavy skill 会通过 `codex` MCP 调 Codex reviewer

也就是说：

- 不是你手工切换到 Codex
- 而是 Claude 在技能流程内部调用它

### 5.5 你现在若要真正启用这条路线，唯一必须先解决的前置条件

先把 Claude 服务恢复到下面这条命令能成功：

```bash
claude -p "Reply with exactly READY" --output-format json --tools ""
```

在这之前，方式 1 只适合做安装准备，不适合正式开跑。

---

## 6. 方式 2：Codex 执行 + Claude reviewer

## 6.1 这条路线当前的现实状态

这条路线当前也不建议你现在就跑，原因同样是：

- reviewer 仍然依赖 Claude 服务

此外还要加一句现实话：

- 当前 `skills-codex-claude-review/*` 不是全覆盖，且比 `skills-codex/*` 主版本略滞后

所以你可以把它理解为：

- **一种节省 Claude 配额的折中路线**
- **不是当前仓库里最完整的一条路**

### 6.2 安装步骤

#### Step 1：安装基础 Codex 技能包

```bash
mkdir -p /data/home/wly/.codex/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/* /data/home/wly/.codex/skills/
```

#### Step 2：再安装 Claude-review overlay

```bash
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/* /data/home/wly/.codex/skills/
```

#### Step 3：安装 reviewer bridge

```bash
mkdir -p /data/home/wly/.codex/mcp-servers/claude-review
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/claude-review/server.py /data/home/wly/.codex/mcp-servers/claude-review/server.py
codex mcp add claude-review -- python3 /data/home/wly/.codex/mcp-servers/claude-review/server.py
```

验证：

```bash
codex mcp list
```

你应该能看到 `claude-review`。

#### Step 4：验证 Claude reviewer 可用

这一步本质仍然会受当前 Claude 服务状态影响。

如果你只是验证本地 bridge 是否注册了，可以先看：

```bash
codex mcp list
```

但如果要真的让 reviewer 返回内容，Claude 服务仍必须先恢复。

### 6.3 运行全流程的方法

进入项目：

```bash
codex -C /data/home/wly/research/aris_demo_project
```

然后在 Codex 会话里分阶段跑：

```text
/idea-discovery "你的具体研究方向" — AUTO_PROCEED: false, compact: true
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
/auto-review-loop "你的主题" — human checkpoint: true
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

或者一键：

```text
/research-pipeline "你的研究方向" — AUTO_PROCEED: false, human checkpoint: true, compact: true
```

### 6.4 reviewer 在这条路线里怎么被调用

这条路线的 reviewer-heavy skill 不再用 `spawn_agent` 第二个 Codex reviewer，而是改为：

- `mcp__claude-review__review_start`
- `mcp__claude-review__review_reply_start`
- `mcp__claude-review__review_status`

长 prompt 使用异步轮询，这一点设计是合理的。

### 6.5 这条路线最重要的现实提醒

当前仓库里，这条路线不是“所有 reviewer 都已经切到 Claude”。

如果你期望的是：

- Codex 做全部执行
- Claude 负责所有 reviewer 阶段

那么当前仓库还没有完全达到这个状态。

你需要接受：

- 现在它更像“Claude 接管 8 个关键 reviewer-heavy 技能”
- 不是整个 runtime 100% 都切换过去

### 6.6 当前什么时候适合用方式 2

只有在下面两个条件同时满足时再考虑：

1. 你的 Claude 服务恢复可用
2. 你明确想节省 Claude 执行额度，把 Claude 尽量只留给 reviewer

---

## 7. 方式 3：纯 Codex，全流程不依赖 Claude

## 7.1 这是你现在最应该先跑的一条路线

因为：

- 你当前 Codex 已配置完成
- `~/.codex/config.toml` 已经可用
- 当前机器上没有必须依赖外部 reviewer bridge 的阻塞

### 7.2 安装步骤

#### Step 1：安装 `skills-codex/*`

```bash
mkdir -p /data/home/wly/.codex/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/* /data/home/wly/.codex/skills/
```

#### Step 2：这条路线不需要额外 reviewer MCP

确认当前 MCP 状态即可：

```bash
codex mcp list
```

你现在本来就没有 MCP，这对方式 3 没关系。

#### Step 3：确认 Codex 配置仍是 `gpt-5.4`

```bash
sed -n '1,80p' /data/home/wly/.codex/config.toml
```

你当前已经是：

- `model = "gpt-5.4"`
- `model_reasoning_effort = "high"`

这就足够开始。

### 7.3 第一次全流程体验的推荐跑法

第一次不要直接追求“超大题目 + 超长实验 + 一夜投顶会”。

建议目标是：

- 跑通一次完整闭环
- 看到 idea、计划、代码、实验、review、改文几个阶段的文件产物

#### Step 1：进入项目

```bash
codex -C /data/home/wly/research/aris_demo_project
```

#### Step 2：先做 idea discovery

推荐命令：

```text
/idea-discovery "你的具体研究方向" — AUTO_PROCEED: false, compact: true
```

如果你愿意给参考论文和基础仓库，推荐更强的写法：

```text
/idea-discovery "改进某方向上的小型基线方法" — AUTO_PROCEED: false, compact: true, ref paper: <论文URL或PDF路径>, base repo: <GitHub仓库URL>
```

你应该重点检查产物：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`

#### Step 3：进入写 code 和部署实验

如果 `EXPERIMENT_PLAN.md` 已产出，就继续：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
```

这一步会尝试：

1. 读取计划
2. 写实验代码
3. 先跑 sanity
4. 调 `/run-experiment`

#### Step 4：因为你当前 GPU 只有 2 号相对空闲，必须限制到 GPU 2

在第一次体验中，请坚持以下原则：

- 只用 GPU 2
- 只允许 1 个并行实验
- 先做短时 sanity run

你的 `CLAUDE.md` / `AGENTS.md` 里已经写了：

- `preferred_gpu: 2`
- `launch_prefix: CUDA_VISIBLE_DEVICES=2`
- `max_parallel_runs: 1`

如果 agent 在部署阶段仍然没有正确限制，你可以直接补一句人工指令：

```text
所有实验只允许使用本机 GPU 2，启动命令前必须带 CUDA_VISIBLE_DEVICES=2，不得占用 0、1、3 号 GPU。
```

#### Step 5：进入自动 review 循环

一旦你拿到初步结果，就运行：

```text
/auto-review-loop "你的主题" — human checkpoint: true, compact: true
```

在方式 3 里，这一步不是调用 Claude，而是：

- Codex 主 agent 做执行
- 第二个 Codex reviewer agent 做审稿

你要重点检查：

- `AUTO_REVIEW.md`
- `REVIEW_STATE.json`
- 是否有新的实验、表格、结果解释被补充

#### Step 6：进入论文写作

当你已经有：

- 初步实验结果
- `AUTO_REVIEW.md`
- `NARRATIVE_REPORT.md`

就可以开始论文写作：

```text
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

如果你还没有把叙事整理进 `NARRATIVE_REPORT.md`，可以先给 Codex 明确任务：

```text
先根据 IDEA_REPORT.md、refine-logs/FINAL_PROPOSAL.md、EXPERIMENT_PLAN.md、AUTO_REVIEW.md 整理一份完整的 NARRATIVE_REPORT.md，然后再进入 /paper-writing。
```

#### Step 7：看论文 reviewer 与修改循环

`/paper-writing` 内部会继续串：

- `/paper-plan`
- `/paper-figure`
- `/paper-write`
- `/paper-compile`
- `/auto-paper-improvement-loop`

你应该重点看：

- `paper/main.tex`
- `paper/main.pdf`
- `PAPER_PLAN.md`
- `PAPER_IMPROVEMENT_LOG.md` 或相关改进日志

#### Step 8：如果你只想体验一次“最像完整科研”的一键命令

在方式 3 下，你也可以直接尝试：

```text
/research-pipeline "你的研究方向" — AUTO_PROCEED: false, human checkpoint: true, compact: true
```

但我仍然建议第一次先分阶段跑，因为更容易定位问题。

### 7.4 方式 3 下最推荐的首次体验顺序

按顺序：

1. `/idea-discovery`
2. `/experiment-bridge`
3. `/auto-review-loop`
4. 整理 `NARRATIVE_REPORT.md`
5. `/paper-writing`

这是你现在最现实的一条试跑路径。

---

## 8. 三条路线的“第一次完整体验”建议命令清单

## 8.1 方式 1：Claude 主线

前置：

```bash
mkdir -p /data/home/wly/.claude/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/* /data/home/wly/.claude/skills/
claude mcp add codex -s user -- codex mcp-server
claude -p "Reply with exactly READY" --output-format json --tools ""
```

如果最后一条成功，再进入：

```bash
cd /data/home/wly/research/aris_demo_project
claude
```

会话内：

```text
/idea-discovery "你的研究方向" — AUTO_PROCEED: false, compact: true
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
/auto-review-loop "你的主题" — human checkpoint: true, compact: true
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

## 8.2 方式 2：Codex 主执行 + Claude reviewer

前置：

```bash
mkdir -p /data/home/wly/.codex/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/* /data/home/wly/.codex/skills/
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/* /data/home/wly/.codex/skills/
mkdir -p /data/home/wly/.codex/mcp-servers/claude-review
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/claude-review/server.py /data/home/wly/.codex/mcp-servers/claude-review/server.py
codex mcp add claude-review -- python3 /data/home/wly/.codex/mcp-servers/claude-review/server.py
codex mcp list
```

然后：

```bash
codex -C /data/home/wly/research/aris_demo_project
```

会话内：

```text
/idea-discovery "你的研究方向" — AUTO_PROCEED: false, compact: true
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
/auto-review-loop "你的主题" — human checkpoint: true
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

但当前你要记住：

- 这条路现在仍然会被 Claude reviewer 服务状态卡住

## 8.3 方式 3：纯 Codex

前置：

```bash
mkdir -p /data/home/wly/.codex/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/* /data/home/wly/.codex/skills/
sed -n '1,80p' /data/home/wly/.codex/config.toml
```

启动：

```bash
codex -C /data/home/wly/research/aris_demo_project
```

会话内推荐顺序：

```text
/idea-discovery "你的研究方向" — AUTO_PROCEED: false, compact: true
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — compact: true
/auto-review-loop "你的主题" — human checkpoint: true, compact: true
/paper-writing "NARRATIVE_REPORT.md" — venue: ICLR, human checkpoint: true
```

或者一键：

```text
/research-pipeline "你的研究方向" — AUTO_PROCEED: false, human checkpoint: true, compact: true
```

---

## 9. 你现在最应该怎么做

如果你的目标是尽快获得一次完整 ARIS 体验，我建议你按下面顺序执行。

### 第一阶段：现在立刻做

1. 创建 demo 项目目录
2. 写好 `CLAUDE.md` 和 `AGENTS.md`
3. 把 `skills-codex/*` 装到 `~/.codex/skills/`
4. 直接走方式 3
5. 分阶段执行：
   - `/idea-discovery`
   - `/experiment-bridge`
   - `/auto-review-loop`
   - `/paper-writing`

### 第二阶段：等 Claude 服务恢复后再做

1. 补方式 1 的主线安装
2. 验证 `claude -p ... READY`
3. 再重跑同一个 demo 项目，对比方式 1 和方式 3 的差异

### 第三阶段：如果你仍然想省 Claude 配额

1. 再尝试方式 2
2. 接受它当前是“部分关键 reviewer 由 Claude 接管”的现实

---

## 10. 最终建议

基于你当前机器状态和当前仓库状态：

- **现在就跑：方式 3**
- **长期最优：方式 1**
- **节流折中：方式 2**

如果你下一步要我继续帮你推进，最合理的动作是：

1. 我直接帮你把方式 3 所需的 `~/.codex/skills` 和 demo 项目骨架准备好。
2. 然后我们就在 `/data/home/wly/research/aris_demo_project` 上真正发起第一次 ARIS 全流程试跑。
