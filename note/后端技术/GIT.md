## init

```shell
git init # 初始化git目录，然后添加文件
git config --global user.name "xxx" 
git config --global user.email "xxx@xxx.com"
git remote add origin git@github.com:loxn/git-demo.git # 连接到远端库
git clone git@github.com:loxn/git-demo.git # 下载代码
```

## add

```shell
git status # 新增或者修改过的文件会显示红色

git add . # 将目录下所有的文件添加到 暂存区
git reset filename # 撤销add操作
```

## commit

```shell
git commit -m 'init commit' # 提交 暂存区 的文件到 本地工作区
git commit -am 'init commit' # add and commit
git commit --amend -m 'change commit info' # 修改commit信息
git reset --soft HEAD^ # 撤销一次commit，或者HEAD~1，--soft，不会撤销 add
git reset --soft HEAD~2 # 撤销两次commit
git reset --mixed HEAD^ # 撤销一次commit，并且撤销 add
git reset --hard HEAD^ # 撤销commit，文件也会 revert
```

## push

```shell
git push origin master # 提交到远端
git push origin [branch_name] # 提交分支到远端
```

## pull

```shell
git pull origin master # 获取远程分支master并merge到当前分支
```

## log

```shell
git log # 查看日志，最新的在最上面
git log -n # 查看最近n条日志
git log --stat # 查看日志，并查看变动文件
git log -p -m # 查看日志，并查看文件变动内容
```

## tag

```sh
git tag	# 查看所有tag
git tag -a v1.0 -m '这是1.0版本' # 新增一个tag
git show v1.0 # 查看tag
git push --tags # 推送所有tag到远端
```

## checkout

```shell
git checkout filename # 从本地工作区还原文件
```

## branch&checkout

```shell
git branch # 查看当前分支
git branch -a # 查看所有分支，本地和远端
git branch -r # 查看远端所有分支
git branch --contains [commit_id] # 查看含有某个commit_id的分支

git branch [branch_name] # 从当前分支创建新分支
git branch -m [branch_name] [new_branch_name] # 修改本地分支的名称
git checkout [branch_name] # 切换到分支
git checkout -b [branch_name] # 从当前分支创建并切到新分支
git checkout -b dev origin/dev # 从远端dev分支创建本地的dev分支
git checkout v2.0  # 切到某个tag
git merge [branch_name] # 合并本地分支到当前分支
git merge origin/master # 合并远端master到本地分支
git branch --merged # 查看已经合并到当前分支的分支
git branch --no-merged # 查看没有合并到当前分支的分支
```

## diff

```shell
git diff [path] # 比较工作区和暂存区
git diff --cached [path] # 比较暂存区和本地库
git diff HEAD [path] # 比较工作区和本地库
git diff [commit-id] # 比较工作区和某次提交
git diff --cached [commit-id] # 比较暂存区和某次提交
git diff [commit-id] [commit-id] # 比较两次提交

git diff --HEAD > [patch_name] # 将工作区和本地库的差异做成补丁，会生成一个文件
git diff > [patch_name] # 将工作区和暂存区的差异做成补丁
git diff --cached > [patch_name] # 将暂存区和本地库的差异做成补丁
git apply [patch_name] # 应用补丁
```

## stash

```shell
git stash save 'test stash'	# 暂存工作区和暂存区的修改，git status看不到任何改动
git list # 查看所有暂存
git stash apply stash@{0} # 应用暂存
```







