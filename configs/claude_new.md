Linux 安装 Claude Code
在 Linux 系统上安装和配置 Claude Code

系统要求
Ubuntu 18.04+、CentOS 7+、Debian 9+ 等主流发行版
Git 2.23+（运行 git -v 检查）
Node.js 18+
安装步骤
0. 安装 Git（如未安装）
# Ubuntu / Debian
sudo apt-get update && sudo apt-get install -y git

# CentOS / RHEL
sudo yum install -y git
1. 安装 Node.js
方式一：使用 fnm 安装 推荐
fnm (Fast Node Manager) 无需 sudo，不污染系统环境。

# 安装 fnm
curl -fsSL https://fnm.vercel.app/install | bash

# 重新加载 shell 配置
source ~/.bashrc

# 安装最新 LTS 版本
fnm install --lts
node -v
方式二：使用 NodeSource（Ubuntu/Debian）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
2. 安装 Claude Code
npm install -g @anthropic-ai/claude-code
3. 配置环境变量
echo 'export ANTHROPIC_AUTH_TOKEN="sk-xwKem33Cb6YekBNdtLWsotzobzUsho6jYzrbeQjAyTtGhlcS"' >> ~/.bashrc
echo 'export ANTHROPIC_BASE_URL="https://api.aipaibox.com/"' >> ~/.bashrc
source ~/.bashrc