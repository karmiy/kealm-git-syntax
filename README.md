# Git 命令

[图解资源](https://zhuanlan.zhihu.com/p/132573100)

## init

git 分为：

- 工作区
- 缓冲区
- 版本仓库

```sh
# 将文件夹置为 git 管理仓库，在工作区执行后会生成 .git 的隐藏文件夹
git init
```

## clone

```sh
# 拉取克隆远程仓库代码到本地
git clone https://gitee.com/xx/xx.git # https 方式
```

## ssh

有时在远程仓库克隆、提交时经常要密码验证，这时可以通过 SSH 的方式将自己的公钥添加到远程仓库

```sh
# C盘/Users/xxx/ 下如果没有 .ssh 文件夹，可以新建一个
# 打开 git bash
git config --global user.name "xxx"
git config --global user.email "xxx@qq.com"
ssh-keygen # 生成，不行可以试试 ssh-keygen -t rsa -C "xxx@qq.com"
# 一路回车
# 如果已经存在，会提示 Overwrite (y/n)，确认 y
# 一路回车，最后可以看到 RSA 图形
# 在 C盘/Users/xxx/.ssh/id_rsa.pub
# 里面有乱码，复制后到仓库
# gitee，设置 => SSH 公钥 => 公钥 Input => 粘贴 => 确定
# github，头像 => setting => SSH and GPG keys => New SSH key => Key Input => 粘贴 => Add SSH key
```

## config

```sh
# 查看配置列表
git config --list

# 全局配置 user
git config --global user.name "xx"
git config --global user.email "xx"
```

## status

```sh
# 查看资源状态，查看工作区、缓存区的文件
git status
```

## add

```sh
# 将工作区的资源放入缓存区
# 添加 index.js
git add ./index.js
# 添加全部
git add .
# 查看当前状态，可用看到工作区未缓存的，和缓存区的文件z
git status
```

## commit

```sh
# 把缓存区的资源，提交到仓库
git commit -m "chore: 添加 index"
```

## branch

```sh
# 新建分支，从当前分支拉，但不切换
git branch <branch-name>
# 查看本地分支
git branch
# 查看远程分支
git branch -r
# 查看本地和远程分支
git branch -a
# 删除本地分支
git branch -D <branch-name>
# 重命名
git branch -m <old-branch-name> <new-branch-name>
```

## checkout

```sh
# 切换当前分支
git checkout XXX
# 从当前本地分支创建一个新本地分支
git checkout -b XXX
```

## merge

```sh
git merge dev # 合并本地 dev 分支过来当前本地分支
git merge origin/dev # 合并远程 dev 分支过来当前本地分支
```

### Fast-forward（commit 记录：dev C2）

当前 status：

- master 分支：C0

- dev 分支：C0（从 master 拉取）

操作：

- dev 修改 a.js， C0 => C2
- git checkout master
- git merge dev
- master 分支 C0 => C2

### No-Fast-forward（commit 记录：master C1 + dev C2）

当前 status：

- master 分支：C0

- dev 分支：C0（从 master 拉取）

操作：

- git checkout master
- master 修改 a.js，C0 => C1
- git checkout dev
- dev 修改 a.js，C0 => C2
- git merge master
- dev 分支记录，会出现按提交时间顺序合并 commit（不论是保留 2 个 commit 还是选择其中 1 个），并新增一个 Merge 的 commit C3（如果有冲突，commit 可重命名），C0 => C1 => C2 => C3（Merge branch 'master' into dev）
- git checkout master
- git merge dev
- master 分支记录同 dev

> 这种场景 git log --oneline --graph 查看日志，可以看到 C1 和 C2 是分叉的

## rebase

当前 status：

- master 分支：C0

- dev 分支：C0（从 master 拉取）

操作：

- git checkout dev
- dev 修改 a.js，C0 => C3
- dev 修改 b.js，C0 => C3 => C4
- git checkout master
- master 修改 a.js，C0 => C1
- master 修改 b.js，C0 => C1 => C2
- git checkout dev
- git rebase master
- 迭代解决冲突文件
- a.js 冲突，解决后 git add . + git rebase --continue
- b.js 冲突，解决后 git add . + git rebase --continue
  - 如果中途想回滚整个 rebase 过程（包括之前 a.js），可以 git rebase --abort
- dev 分支记录，会以 master 为基准，dev 新增的 commit C3、C4 会被删除，并复制一份 C3'、C4' 顺序跟随 master 基准，C0 => C1 => C2 => C3' => C4'
- git checkout master
- master 修改 a.js，C0 => C1 => C2 => C5
- git checkout dev
- git rebase master
- dev 分支记录，C0 => C1 => C2 => C5 => C3' => C4'（可以看到以 master 为基准，master 即使后面新增的分支也是在前面）
- 推送 push
  - git push --force origin dev：单人开发，远端 origin 的新增内容可以被覆盖时可用（提交到当前本地分支如何，提交后远程分支也会如何，不适合多人开发，会覆盖别人代码）
  - git push --force-with-lease origin dev：多人开发，该命令强制覆盖前也会金霞一次检查，若该分支有提交会警告

> git log --oneline --graph 查看日志，可用看到提交记录没有分叉，更为清晰，以 master 当前为基准的 commit，只要是 master 上没有的 dev 提交记录，都会被排后面

## rebase -i

交互式变基，可用对提交进行修改

- reword：修改提交信息
- edit：修改此提交
- squash：将提交融合到前一个提交中
- fixup：将提交融合到前一个提交中，不保留该提交的日志消息
- exec：在每个提交上运行我们想要 rebase 的命令
- drop：移除该提交

如编辑某条 commit：

- 提交 C1、C2、C3
- git rebase -i 会出现界面，也可以 git rebase -i HEAD~2 操作最近 2 条记录
- 按 s 等键位进入可编辑，3 条 pick xxxx 里修改第二条 pick 为 e
- ESC 退出编辑，:wq 保存
- git commit --amend
- 按 s 等键位进入可编辑，修改 commit message
- ESC 退出编辑，:wq 保存
- git rebase --continue
- git log --oneline --graph 可用看到 C2 的 commit message 日志已变更

## log

```bash
# 查看日志，commit hash/author/date/commit message
git log
# 图表的形式看到日志记录，更直观
git log --oneline --graph
```

## reset

### 软重置

回退到之前某次 commit，后面 commit 的文件会回到工作区

```sh
git reset --soft HEAD~2 # 指针往前走 2 次 commit
git reset --soft 250c926 # 到某个 commit
```

### 硬重置

与软重置不同的是，后面的 commit 都会消失

```sh
git reset --hard HEAD~2
git reset --hard 250c926
```

> reset 也可以逆向，即 A => B => C => D，先 reset 到 B，如果记住了 D 的 hash，还是可以 reset 回 D，不代表只能回到向下的

## revert

还原某次 commit 的操作，还会创建一个新提交，即只撤销改动的代码，但是 commit 记录还在，修改提交后再新增一条 commit

```sh
git revert 3bc9f8e
git revert 3bc9f8e...4ku9z3m ... # 回滚多次，这是个前开后闭的区间
git revert 3bc9f8e^...4ku9z3m ... # 回滚多次，左右都闭合
git revert --no-commit 3bc9f8e # 回滚后不创建一个新提交，而是可以到时跟当前本地还没提交到代码一起提交
```

## cherry-pick

拉取某条 commit 到本分支

```sh
git cherry-pick 3bc9f8e
# 多个，可以是个区间，左开右闭
git cherry-pick 3bc9f8e...4ku9z3m
# 左右都闭合
git cherry-pick 3bc9f8e^...4ku9z3m
```

## fetch

拉取 origin 的代码更新

```sh
git fetch
git fetch origin master
```

## pull

等于 git fetch + git merge

```sh
git pull
git pull origin feat-test # git pull <远程主机名><远程分支名>:<本地分支名>
git pull --rebase # 以 rebase 的形式拉取
```

## push

```sh
git push # 提交到远程仓库
```

## stash

```sh
# 有时开发一半临时要去别的分支改东西，又不想 commit，就可以暂存
git stash # 把本地改动暂存
git stash save 'feat: 更新弹框'
git stash pop # 应用最近的一次暂存修改，并删除暂存的记录
git stash apply # 应用某次暂存，但不会从暂存列表删除，默认使用暂存的第一个 stash@{0}
git stash apply stash@{$num} # 指定应用的暂存
git stash list # 暂存列表
git stash clear # 删除所有暂存
```

## remote

```sh
git remote update origin --prune # 更新分支列表
```

## alias

可以对一些操作设置别名，如 checkout 太长了，可以配置为 co，这样即可 ` git co dev` 简写操作

```sh
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
```

可参考：

```sh
[alias]
st = status -sb
co = checkout
br = branch
mg = merge
ci = commit
ds = diff --staged
dt = difftool
mt = mergetool
last = log -1 HEAD
latest = for-each-ref --sort=-committerdate --format=\"%(committername)@%(refname:short) [%(committerdate:short)] %(contents)\"
ls = log --pretty=format:\"%C(yellow)%h %C(blue)%ad %C(red)%d %C(reset)%s %C(green)[%cn]\" --decorate --date=short
hist = log --pretty=format:\"%C(yellow)%h %C(red)%d %C(reset)%s %C(green)[%an] %C(blue)%ad\" --topo-order --graph --date=short
type = cat-file -t
dump = cat-file -p
lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

## reflog

可以展示已经执行过的所有动作的日志，包括合并、重置、还原，基本上包含你对你的分支所做的任何修改

这样在操作不当时可用 reset 到对应位置

```sh
git reflog
```

```sh
46f7dce (HEAD -> dev) HEAD@{0}: commit: chore: revert 16
3e20b40 HEAD@{1}: commit: chore: r dev-a17
3bc9f8e (origin/dev) HEAD@{2}: commit: chore: r dev-a16
250c926 HEAD@{3}: reset: moving to 250c926
19ed287 HEAD@{4}: commit: chore: r dev-a15 软重置提交
250c926 HEAD@{5}: reset: moving to 250c926
9d04d77 HEAD@{6}: rebase -i (finish): returning to refs/heads/dev
9d04d77 HEAD@{7}: rebase -i (pick): chore: r dev-a16
250c926 HEAD@{8}: rebase -i (squash): chore: r dev-a14/15
cf37694 HEAD@{9}: rebase -i (start): checkout HEAD~3
0a6d828 HEAD@{10}: rebase -i (finish): returning to refs/heads/dev
0a6d828 HEAD@{11}: rebase -i (pick): chore: r dev-a16
8977155 HEAD@{12}: commit (amend): chore: r dev-a15 修正
94292e8 HEAD@{13}: commit (amend): chore: r dev-a15
0ade3c7 HEAD@{14}: rebase -i: fast-forward
```

```sh
git reset HEAD@{3}
```

## .gitignore

git 忽略列表，列表中文件不会被 git 作为管理文件，也就不会让你提交到缓存区、仓库

```sh
*.txt
node_modules
# 根目录下的 dist
/dist
```



