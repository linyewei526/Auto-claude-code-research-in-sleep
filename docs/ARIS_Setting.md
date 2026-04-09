## 环境配置
### <font style="color:rgba(0, 0, 0, 0.9);">建立 Fork + Upstream（只做一次）</font>
1. Github网页fork原仓库[https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep)（取消勾选<font style="color:rgba(0, 0, 0, 0.9);">Copy the main branch only</font>）
2. 克隆自己的fork到服务器

```bash
cd ARIS
git clone git@github.com:linyewei526/Auto-claude-code-research-in-sleep.git
cd Auto-claude-code-research-in-sleep
```

3. 添加原仓库作为upstream（上游）

```bash
git remote add upstream git@github.com:wanshuiyin/Auto-claude-code-research-in-sleep.git
git remote -v //验证配置
'''
显示
origin  git@github.com:linyewei526/Auto-claude-code-research-in-sleep.git (fetch)
origin  git@github.com:linyewei526/Auto-claude-code-research-in-sleep.git (push)
upstream        git@github.com:wanshuiyin/Auto-claude-code-research-in-sleep.git (fetch)
upstream        git@github.com:wanshuiyin/Auto-claude-code-research-in-sleep.git (push)
'''
```

### <font style="color:rgba(0, 0, 0, 0.9);">创建 Worktree（多分支并行编辑）</font>
1. 查看当前分支

```bash
git branch -r //查看所有可用分支
'''
输出
  origin/HEAD -> origin/main
  origin/aris-code
  origin/feature/paper-poster
  origin/feature/paper-slides
  origin/main
'''
```

2. 创建各个分支的独立工作目录

```bash
git worktree add ../aris-code origin/aris-code
'''
显示
Preparing worktree (detached HEAD 7ce14ed)
HEAD is now at 7ce14ed Update What\'s New to v0.3.3 (CN)
'''
git worktree add ../paper-poster origin/feature/paper-poster
'''
显示
Preparing worktree (detached HEAD a430df6)
HEAD is now at a430df6 Merge PR #44: paper-poster skill + resolve README conflicts
'''
git worktree add ../paper-slides origin/feature/paper-slides
'''
显示
Preparing worktree (detached HEAD d23e3cd)
HEAD is now at d23e3cd Merge PR #42: paper-slides skill + resolve README conflicts
'''
```

3. 查看所有工作目录

```bash
git worktree list
'''
显示
/data/home/wly/ARIS/Auto-claude-code-research-in-sleep  d86b656 [main]
/data/home/wly/ARIS/aris-code                           7ce14ed (detached HEAD)
/data/home/wly/ARIS/paper-poster                        a430df6 (detached HEAD)
/data/home/wly/ARIS/paper-slides                        d23e3cd (detached HEAD)
'''
```

4. 把 detached HEAD 转换为跟踪分支

```bash
# 进入 aris-code 目录
cd ~/ARIS/aris-code

# 查看当前状态
git status
# 显示：HEAD detached at 7ce14ed或Not currently on any branch.

# 建立本地分支跟踪 origin/aris-code
git checkout -b aris-code origin/aris-code
'''
显示
branch \'aris-code\' set up to track \'origin/aris-code\'.
Switched to a new branch \'aris-code\'
'''

# 验证
git branch -vv
'''
显示
* aris-code 7ce14ed [origin/aris-code] Update What\'s New to v0.3.3 (CN)
+ main      d86b656 (/data/home/wly/ARIS/Auto-claude-code-research-in-sleep) 
[origin/main] docs: fix meta-optimize setup instructions — run in normal terminal, 
not Claude Code
'''

# 同理修复其他worktree
cd ~/ARIS/paper-poster
git checkout -b feature/paper-poster origin/feature/paper-poster

cd ~/ARIS/paper-slides  
git checkout -b feature/paper-slides origin/feature/paper-slides

# 最终验证
cd ~/ARIS/Auto-claude-code-research-in-sleep
git worktree list
'''
显示
/data/home/wly/ARIS/Auto-claude-code-research-in-sleep  d86b656 [main]
/data/home/wly/ARIS/aris-code                           7ce14ed [aris-code]
/data/home/wly/ARIS/paper-poster                        a430df6 [feature/paper-poster]
/data/home/wly/ARIS/paper-slides                        d23e3cd [feature/paper-slides]
'''
```

### <font style="color:rgba(0, 0, 0, 0.9);">新建分支</font>
1. 在服务器端建立新分支/文件夹

```bash
# 进入主干仓库（推荐从这里创建，管理方便）
cd ~/ARIS/Auto-claude-code-research-in-sleep

# 从 main 创建你的个性化分支（也可以用 aris-code 为基础）
git checkout -b personal main

# 验证
git branch -vv
'''
显示
+ aris-code            7ce14ed (/data/home/wly/ARIS/aris-code) [origin/aris-code] Update What\'s New to v0.3.3 (CN)
+ feature/paper-poster a430df6 (/data/home/wly/ARIS/paper-poster) [origin/feature/paper-poster] Merge PR #44: paper-poster skill + resolve README conflicts
+ feature/paper-slides d23e3cd (/data/home/wly/ARIS/paper-slides) [origin/feature/paper-slides] Merge PR #42: paper-slides skill + resolve README conflicts
  main                 d86b656 [origin/main] docs: fix meta-optimize setup instructions — run in normal terminal, not Claude Code
* personal             d86b656 docs: fix meta-optimize setup instructions — run in normal terminal, not Claude Code
'''
```

2. 此时服务器端ARIS 目录中没有 personal 文件夹，但是工作目录 ~/ARIS/Auto-claude-code-research-in-sleep 已经切换到了 personal 分支

```bash
# 验证当前所在分支
git branch --show-current
# 输出：personal

# 查看文件，和 main 完全一样（因为刚分出）
ls
```

3. 创建独立 worktree（推荐，和其他分支一致）

```bash
cd ~/ARIS/Auto-claude-code-research-in-sleep

# 回到 main
git checkout main

# 创建 personal worktree（基于 origin/personal，如果已 push）
git push -u origin personal  # 先推送
git worktree add ../personal origin/personal

# 验证
git worktree list
'''
显示
/data/home/wly/ARIS/Auto-claude-code-research-in-sleep  d86b656 [main]
/data/home/wly/ARIS/aris-code                           7ce14ed [aris-code]
/data/home/wly/ARIS/paper-poster                        a430df6 [feature/paper-poster]
/data/home/wly/ARIS/paper-slides                        d23e3cd [feature/paper-slides]
/data/home/wly/ARIS/personal                            d86b656 (detached HEAD)
'''
```

4. 修复 personal worktree

```bash
cd ~/ARIS/personal

# 直接切换到已有的 personal 分支（建立跟踪）
git checkout personal

# 验证
git branch -vv
'''
显示
+ aris-code            7ce14ed (/data/home/wly/ARIS/aris-code) [origin/aris-code] Update What\'s New to v0.3.3 (CN)
+ feature/paper-poster a430df6 (/data/home/wly/ARIS/paper-poster) [origin/feature/paper-poster] Merge PR #44: paper-poster skill + resolve README conflicts
+ feature/paper-slides d23e3cd (/data/home/wly/ARIS/paper-slides) [origin/feature/paper-slides] Merge PR #42: paper-slides skill + resolve README conflicts
+ main                 d86b656 (/data/home/wly/ARIS/Auto-claude-code-research-in-sleep) [origin/main] docs: fix meta-optimize setup instructions — run in normal terminal, not Claude Code
* personal             d86b656 [origin/personal] docs: fix meta-optimize setup instructions — run in normal terminal, not Claude Code
'''

# 验证 worktree 状态
cd ~/ARIS/Auto-claude-code-research-in-sleep
git worktree list
'''
显示
/data/home/wly/ARIS/Auto-claude-code-research-in-sleep  d86b656 [main]
/data/home/wly/ARIS/aris-code                           7ce14ed [aris-code]
/data/home/wly/ARIS/paper-poster                        a430df6 [feature/paper-poster]
/data/home/wly/ARIS/paper-slides                        d23e3cd [feature/paper-slides]
/data/home/wly/ARIS/personal                            d86b656 [personal]
'''
```

### <font style="color:rgba(0, 0, 0, 0.9);">github提交</font>
1. 确认当前状态

```bash
cd ~/ARIS/personal

# 查看状态（示例：大量删除）
git status
# 预期输出：
# On branch personal
# Changes not staged for commit:
#   deleted:    xxx.file
#   deleted:    yyy.file
#   ...
```

2. 提交操作

```bash
# 添加到暂存区
git add .

# 提交
git commit -m "示例: 清空 personal 分支，准备添加个人内容"

# 推送到 GitHub
git push origin personal
```



