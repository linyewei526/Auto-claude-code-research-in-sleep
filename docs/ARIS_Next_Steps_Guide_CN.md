# ARIS 下一步配置与使用实操指南（基于你当前服务器状态）

## 1. 你的当前状态

根据你现有的：

- `/data/home/wly/ARIS/personal/docs/ARIS_Setting.md`

以及仓库真实状态，你现在已经完成了这些事：

1. 你已经在 GitHub 上 fork 了上游仓库。
2. 你已经把 fork clone 到服务器。
3. 你已经把多个分支用 `git worktree` 形式展开成：
   - `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`
   - `/data/home/wly/ARIS/aris-code`
   - `/data/home/wly/ARIS/paper-poster`
   - `/data/home/wly/ARIS/paper-slides`
   - `/data/home/wly/ARIS/personal`
4. 你已经有一个 `personal` 分支专门存放自己的记录。

**但你还没有真正进入 ARIS 的“运行态配置阶段”。**

也就是说，你完成的是：

- Git 层面的准备

你还没完成的是：

- 执行器安装
- reviewer 安装
- skills 安装
- MCP 配置
- 实际科研项目目录初始化
- `CLAUDE.md` 配置
- 首次运行

---

## 2. 先做一个最重要的选择：你准备走哪条路线

你现在有三条路可选。

### 路线 A：主线标准路线，推荐优先

组合：

- 执行者：Claude Code
- 审稿者：Codex reviewer（默认）或其他 reviewer
- 技能来源：`Auto-claude-code-research-in-sleep/skills`

优点：

- 与主线 README 完全一致
- 文档最丰富
- 工作流最完整
- 最适合你当前“先学会怎么用 ARIS”

缺点：

- 需要 Claude Code 可用

### 路线 B：Codex 原生路线

组合：

- 执行者：Codex CLI
- 技能来源：`skills/skills-codex/`
- 审稿者：第二个 Codex、Claude bridge、Gemini bridge 都可以

优点：

- 你已经有 Codex 账号时更容易起步

缺点：

- 虽然仓库支持，但它不是主线文档默认路径
- 如果只用 Codex 自己审自己，跨模型对抗性会下降

### 路线 C：`aris-code` 独立 CLI

组合：

- 执行者/审稿者都由 `aris` 独立程序管理

优点：

- 结构完整

缺点：

- 目前不建议你把它作为 Linux 服务器上的第一落地方案
- 你现在的主要目标是“把 ARIS 用起来”，不是研究第二套运行时

### 建议

**建议你优先走路线 A。**

如果你暂时没有 Claude Code 可用，再走路线 B。

---

## 3. 我给你的实际建议

### 3.1 最推荐的组合

如果你有条件，推荐：

- 执行者：Claude Code
- 审稿者：Codex reviewer
- 工具仓库：`/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`
- 你的实际科研项目目录：另建，不放在 ARIS 仓库里面

### 3.2 如果你现在只有 Codex 账号

你依然可以使用 ARIS，但要分情况：

#### 情况 1：你只有 Codex，没有 Claude Code

可以：

- 走 `skills/skills-codex/` 路线
- 让 Codex 作为执行者

但这时你还需要决定 reviewer：

- reviewer 也用 Codex：可以用，但变成同模型体系
- reviewer 用 Gemini：需要 Gemini API Key
- reviewer 用 Claude：需要 Claude Code CLI 可用

#### 情况 2：你有 Codex，同时也能登录 Claude Code

这是最理想的主线组合：

- Claude Code 执行
- Codex 审稿

---

## 4. 正确的目录使用方式

从现在开始，请你明确区分两个目录类别。

### 4.1 工具仓库目录

这几个目录是工具仓库，不是你的科研项目本体：

- `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`
- `/data/home/wly/ARIS/aris-code`
- `/data/home/wly/ARIS/paper-poster`
- `/data/home/wly/ARIS/paper-slides`

### 4.2 你的真实科研项目目录

推荐你单独创建：

```bash
mkdir -p /data/home/wly/research
mkdir -p /data/home/wly/research/aris_demo_project
```

以后你真正运行 ARIS 的位置应是：

- `/data/home/wly/research/aris_demo_project`

而不是：

- `/data/home/wly/ARIS/Auto-claude-code-research-in-sleep`

原因：

- ARIS 仓库是“工具”
- `aris_demo_project` 才是“你的研究项目”

---

## 5. 路线 A：主线标准路线，详细步骤

下面默认你采用：

- 执行者：Claude Code
- 审稿者：Codex reviewer

### Step 0：检查你当前工具仓库是否正常

```bash
cd /data/home/wly/ARIS/Auto-claude-code-research-in-sleep
git branch --show-current
git remote -v
git worktree list
```

你应该看到：

- 当前分支是 `main`
- 有 `origin` 和 `upstream`
- worktree 包含 `aris-code`、`paper-poster`、`paper-slides`、`personal`

### Step 1：安装基础依赖

以下命令按 Linux 服务器准备。

#### 1.1 Node.js 和 npm

如果系统还没有：

```bash
node -v
npm -v
```

没有的话，按你的系统安装。Ubuntu 常见方式例如：

```bash
sudo apt update
sudo apt install -y nodejs npm
```

如果系统源版本太老，建议你自己按 nvm 或 NodeSource 安装更新版本。

#### 1.2 Python 3 和 pip

```bash
python3 --version
pip3 --version
```

没有的话：

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

#### 1.3 可选但常用：`screen`、`tmux`、`rsync`

```bash
screen -v
tmux -V
rsync --version
```

没有的话：

```bash
sudo apt install -y screen tmux rsync
```

#### 1.4 如果将来要写论文，再装 LaTeX

如果你现在只是先跑 workflow 1 和 2，这一步可先跳过。

后续要用 `/paper-writing` 时再装：

```bash
sudo apt install -y texlive-full latexmk poppler-utils
latexmk --version
pdfinfo -v
```

### Step 2：安装 Claude Code

这一步取决于你自己的使用方式与账号状态。你需要确保在这台服务器上可以启动 `claude`。

先验证：

```bash
which claude
claude --help
```

如果当前服务器上还没有 Claude Code，请按你自己的安装方式完成安装与登录。你最终至少要达到：

```bash
claude --help
```

可以正常工作。

### Step 3：安装 Codex CLI

主线默认 reviewer 要用它。

```bash
npm install -g @openai/codex
codex --help
```

然后做初始化：

```bash
codex setup
```

如果 `codex setup` 会提示选择模型，按主线推荐优先选：

- `gpt-5.4`

然后确认本地配置里 reviewer 模型正确，例如：

```bash
grep -n 'model' ~/.codex/config.toml
```

如果你看到：

```toml
model = "gpt-5.4"
```

就符合主线默认建议。

### Step 4：把 Codex 注册成 Claude Code 的 MCP server

在服务器上执行：

```bash
claude mcp add codex -s user -- codex mcp-server
```

然后检查 Claude Code 的配置中是否已有对应记录。具体文件位置可能因版本变化而不同，但一般可用：

```bash
claude mcp list
```

如果列出了 `codex`，说明接入成功。

### Step 5：安装 ARIS 主线技能到 Claude Code

在服务器上执行：

```bash
mkdir -p ~/.claude/skills
cp -r /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/* ~/.claude/skills/
```

如果以后你要更新技能，可以再次执行：

```bash
cp -r /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/* ~/.claude/skills/
```

### Step 6：建立真实科研项目目录

不要在 ARIS 仓库里做实验。创建独立项目目录：

```bash
mkdir -p /data/home/wly/research/aris_demo_project
cd /data/home/wly/research/aris_demo_project
```

建立最基础的目录结构：

```bash
mkdir -p docs refine-logs papers figures logs
```

### Step 7：给项目写 `CLAUDE.md`

这是最关键的一步。

在你的研究项目目录里创建：

```bash
cat > /data/home/wly/research/aris_demo_project/CLAUDE.md << 'EOF'
# CLAUDE.md

## Project

- Project name: ARIS demo project
- Goal: 用 ARIS 从文献调研开始，跑通一个最小科研流程
- Language: 中文输出优先

## Pipeline Status

stage: idea-discovery
idea: "尚未确定"
contract: docs/research_contract.md
current_branch: main
baseline: "尚未建立"
training_status: "none"
active_tasks:
  - "none"
next: "先完成 research brief，然后运行 /idea-discovery"

## GPU Environment

- gpu: local
- 这台机器有直接 GPU 访问；如果没有 GPU，请把 gpu 改成 remote / vast / modal
- Python env activate: `conda activate research`
- code directory: `/data/home/wly/research/aris_demo_project`

## Research Rules

- 所有关键阶段都要更新 Pipeline Status
- 优先把重要结论写入 docs/research_contract.md、EXPERIMENT_LOG.md、findings.md
- 如果 context 被压缩或开启新会话，先读 CLAUDE.md，再读 docs/research_contract.md
EOF
```

如果你现在没有本地 GPU，而是用远程服务器，把 `GPU Environment` 改成类似：

```markdown
## Remote Server

- gpu: remote
- SSH: `ssh my-gpu-server`
- conda env: `research`
- activate: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- code directory: `/home/wly/projects/aris_demo_project`
- run backend: `screen`
```

### Step 8：准备研究输入文件

建议你从模板开始，不要空跑。

先复制研究简报模板：

```bash
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_BRIEF_TEMPLATE.md \
  /data/home/wly/research/aris_demo_project/RESEARCH_BRIEF.md
```

再编辑它，写上：

- 你想研究的问题
- 领域范围
- 你已有的约束
- 你不想做的方向
- 可用算力

如果你已经有论文 PDF，也可以放到：

```bash
mkdir -p /data/home/wly/research/aris_demo_project/papers
```

然后放入：

- 参考论文 PDF
- baseline 论文 PDF

### Step 9：启动 Claude Code，进入项目目录

```bash
cd /data/home/wly/research/aris_demo_project
claude
```

进入后，先别急着全流程自动跑。第一次建议按下面顺序做。

### Step 10：第一次使用时的推荐命令顺序

#### 10.1 只做文献调研

```text
/research-lit "你的研究主题"
```

例如：

```text
/research-lit "离散扩散语言模型中的 factorized gap"
```

这一步的目的不是立刻产出论文，而是确认：

- skills 是否真的被加载
- WebSearch 是否能工作
- Claude Code 是否能读项目目录

#### 10.2 运行完整 idea discovery

```text
/idea-discovery "离散扩散语言模型中的 factorized gap"
```

如果你已经写好了 `RESEARCH_BRIEF.md`，这个命令会自动把它当成详细上下文。

你应该重点观察是否产出：

- `IDEA_REPORT.md`
- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/EXPERIMENT_PLAN.md`
- `refine-logs/EXPERIMENT_TRACKER.md`

#### 10.3 如果你要手动保守一点

可以这样运行：

```text
/idea-discovery "离散扩散语言模型中的 factorized gap" — AUTO_PROCEED: false
```

这样在选 idea 时不会自动继续。

### Step 11：进入实验桥接

当你确认某个 idea 可行后，执行：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md"
```

如果你要更保守：

```text
/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — AUTO_DEPLOY: false
```

这样它会先实现与审查代码，但不会自动部署。

### Step 12：进入自动 review loop

当你已经拿到初步实验结果后，再运行：

```text
/auto-review-loop "当前研究主题"
```

例如：

```text
/auto-review-loop "离散扩散语言模型中的 factorized gap"
```

更推荐你第一次这样跑：

```text
/auto-review-loop "离散扩散语言模型中的 factorized gap" — human checkpoint: true
```

这样每轮 reviewer 出分后你可以人工看一眼，不至于完全黑箱。

### Step 13：进入论文写作

当你已经有研究叙事以后，先从模板生成叙事报告：

```bash
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/NARRATIVE_REPORT_TEMPLATE.md \
  /data/home/wly/research/aris_demo_project/NARRATIVE_REPORT.md
```

然后填充你的：

- claims
- experiments
- results
- figure descriptions

再在 Claude Code 中运行：

```text
/paper-writing "NARRATIVE_REPORT.md"
```

### Step 14：如果你想一口气全流程

当你已经熟悉前面步骤，再考虑：

```text
/research-pipeline "你的研究方向"
```

第一次不建议上来就这么干。更稳妥的方式是：

1. 先单独跑 `/research-lit`
2. 再跑 `/idea-discovery`
3. 再跑 `/experiment-bridge`
4. 再跑 `/auto-review-loop`

---

## 6. 可选增强配置

### 6.1 会话恢复与状态持久化

这是非常推荐你尽快加上的能力。

它的核心不是脚本，而是你要在项目 `CLAUDE.md` 中持续维护：

- `## Pipeline Status`

你可以再进一步按仓库文档里的方式加 hooks，但第一步先把 `CLAUDE.md` 维护好。

### 6.2 启用 Meta Optimize 日志

如果你想以后分析自己怎么使用 ARIS，可以在普通终端中做一次性设置：

```bash
cd /data/home/wly/research/aris_demo_project
mkdir -p .claude .aris/meta tools/meta_opt
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/claude-hooks/meta_logging.json .claude/settings.json
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/tools/meta_opt/*.sh tools/meta_opt/
chmod +x tools/meta_opt/*.sh
```

然后再启动 `claude`。

### 6.3 如果你要接 Zotero

```bash
uv tool install zotero-mcp-server
claude mcp add zotero -s user -- zotero-mcp -e ZOTERO_LOCAL=true
```

### 6.4 如果你要接 Obsidian

```bash
claude mcp add obsidian-vault -s user -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

### 6.5 如果你要用 watchdog 监控远程实验

把脚本同步到服务器后：

```bash
screen -dmS watchdog python3 tools/watchdog.py
```

注册任务的方式见主线 `WATCHDOG_GUIDE_CN.md`。

---

## 7. 路线 B：如果你暂时没有 Claude Code，只想用 Codex

如果你暂时只有 Codex，那么可以走 Codex 原生技能路线。

### Step 1：安装 Codex 技能

```bash
mkdir -p ~/.codex/skills
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex/* ~/.codex/skills/
```

### Step 2：进入你的研究项目目录

```bash
cd /data/home/wly/research/aris_demo_project
```

### Step 3：启动 Codex，并在项目目录中使用技能

这一步的具体命令形式取决于你当前安装的 Codex CLI 版本与工作模式，但核心思想是：

- 使用 `~/.codex/skills` 中的 ARIS Codex 技能
- 仍然在你的研究项目目录中运行，而不是在 ARIS 仓库中运行

### Step 4：决定 reviewer 怎么配

你有 3 种主要选择。

#### 方案 B1：Codex 自己当 reviewer

优点：

- 最简单

缺点：

- 不是跨模型
- reviewer 独立性较弱

#### 方案 B2：Codex + Claude reviewer overlay

安装：

```bash
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-claude-review/* ~/.codex/skills/
mkdir -p ~/.codex/mcp-servers/claude-review
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/claude-review/server.py ~/.codex/mcp-servers/claude-review/server.py
codex mcp add claude-review -- python3 ~/.codex/mcp-servers/claude-review/server.py
```

这要求你本机能用 `claude`。

#### 方案 B3：Codex + Gemini reviewer overlay

安装：

```bash
cp -a /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/skills/skills-codex-gemini-review/* ~/.codex/skills/
mkdir -p ~/.codex/mcp-servers/gemini-review
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/mcp-servers/gemini-review/server.py ~/.codex/mcp-servers/gemini-review/server.py
codex mcp add gemini-review --env GEMINI_REVIEW_BACKEND=api -- python3 ~/.codex/mcp-servers/gemini-review/server.py
```

这要求你有：

- `GEMINI_API_KEY` 或 `GOOGLE_API_KEY`

---

## 8. 你是否需要外部 API / 账号

这是你问题里最关键的一点，我直接回答结论。

### 8.1 简短结论

**需要。**

但不是“必须是 Claude + 必须是 OpenAI”。

真正必须的是：

- 至少一个可用的执行者模型后端
- 最好再有一个独立的 reviewer 模型后端

### 8.2 你现在有 Codex 账号，够不够

#### 只问“能不能启动 ARIS 某种形态”

可以。

你可以：

- 走 Codex 原生技能路线
- 用 `skills/skills-codex/`

#### 如果问“能不能按主线默认方式使用”

不完全够。

因为主线默认方式是：

- Claude Code 执行
- Codex 审稿

所以你还需要：

- Claude Code 可用

#### 如果问“必须额外买 Claude 吗”

不一定。

因为仓库已经提供替代路线：

- GLM + GPT
- GLM + MiniMax
- 任意执行器 + 任意 OpenAI-compatible reviewer
- ModelScope 免费路线
- Codex + Gemini reviewer
- Codex + Claude reviewer

### 8.3 不同路线对应的账号/API 需求

| 路线 | 需要什么 |
|---|---|
| 主线默认：Claude Code + Codex reviewer | Claude Code 可用 + Codex 可用 |
| Codex 原生技能，Codex 自审 | 只要 Codex |
| Codex + Gemini reviewer | Codex + Gemini API Key |
| Codex + Claude reviewer | Codex + Claude Code 可用 |
| ModelScope 免费路线 | 一个 ModelScope Key 即可覆盖执行者和 reviewer |
| GLM + MiniMax | GLM Key + MiniMax Key |
| `aris-code` 独立 CLI | 至少一个 executor provider key；如果要 reviewer，再来一个 reviewer key |

### 8.4 从效果角度的建议

如果你想尽量接近 ARIS 设计初衷，推荐优先级如下：

1. **Claude Code + Codex reviewer**
2. **Codex + Gemini reviewer**
3. **ModelScope 双模型路线**
4. **Codex 单模型自审**

原因：

- ARIS 的核心收益来自跨模型对抗，不是单模型自循环

### 8.5 对你最现实的建议

如果你现在只有 Codex，可按下面顺序考虑：

1. 先确认你能不能在服务器上使用 Claude Code
2. 如果能，就直接走主线默认路线
3. 如果不能，再决定：
   - 只用 Codex 先跑通最小流程
   - 或者补一个 Gemini / ModelScope reviewer

---

## 9. 我建议你的实际执行顺序

如果你想最稳地把事情推进，按下面顺序做。

### 第一阶段：先把环境跑通

1. 安装 Claude Code
2. 安装 Codex CLI
3. `claude mcp add codex -s user -- codex mcp-server`
4. 把 `skills/*` 拷到 `~/.claude/skills/`
5. 新建独立研究项目目录
6. 写好项目 `CLAUDE.md`

### 第二阶段：跑最小闭环

1. `/research-lit "你的主题"`
2. `/idea-discovery "你的主题" — AUTO_PROCEED: false`
3. 检查 `IDEA_REPORT.md` 和 `refine-logs/`

### 第三阶段：再碰实验

1. `/experiment-bridge "refine-logs/EXPERIMENT_PLAN.md" — AUTO_DEPLOY: false`
2. 确认代码生成逻辑没问题
3. 再开始部署实验

### 第四阶段：再碰自动 review loop

1. `/auto-review-loop "你的主题" — human checkpoint: true`

### 第五阶段：最后再碰全流程自动化

1. `/research-pipeline "你的主题"`
2. `/paper-writing "NARRATIVE_REPORT.md"`

---

## 10. 你现在最应该做的第一批命令

如果你要立刻继续，最实用的是下面这一组。

### 10.1 工具仓库层

```bash
cd /data/home/wly/ARIS/Auto-claude-code-research-in-sleep
git pull
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/
```

### 10.2 reviewer 层

```bash
npm install -g @openai/codex
codex setup
claude mcp add codex -s user -- codex mcp-server
```

### 10.3 项目层

```bash
mkdir -p /data/home/wly/research/aris_demo_project
cd /data/home/wly/research/aris_demo_project
mkdir -p docs refine-logs papers figures logs
cp /data/home/wly/ARIS/Auto-claude-code-research-in-sleep/templates/RESEARCH_BRIEF_TEMPLATE.md RESEARCH_BRIEF.md
```

然后创建 `CLAUDE.md`，再启动：

```bash
claude
```

第一次进入后先跑：

```text
/research-lit "你的研究主题"
```

---

## 11. 最后一句建议

你现在最容易犯的错误不是“命令不会敲”，而是：

- 把 ARIS 工具仓库当成科研项目目录
- 上来就想直接全自动跑全流程
- 没有 `CLAUDE.md` 就开始用实验工作流
- 只有一个模型却误以为已经拥有完整的默认 ARIS 架构

正确姿势是：

- 工具仓库和研究项目目录分离
- 先搭执行者和 reviewer
- 先跑最小工作流
- 再逐步扩到自动实验、自动 review、自动写论文

