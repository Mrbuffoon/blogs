
# 1、git常用命令速查
### **1.1 初始化相关操作**
```
$ git config --global user.name "John Doe" # 配置用户名 ！仅第一次必须

$ git config --global user.email je@example.com # 配置电邮 ！仅第一次必须

$ git config --list # 查看配置信息

$ git init # 初始化仓库

$ git remote add origin git@github.com:michaelliao/learngit.git # 将本地仓库与远程仓库关联(远程仓库已创建)
```
### 1.2 提交代码相关操作
```
$ git clone git@github.com:michaelliao/learngit.git # 从远程仓库克隆代码到本地

$ git add [文件名/./--all] # 添加新的文件到暂存区

$ git commit -m "Story 182: Fix benchmark" # 提交更新

$ git commit -a -m 'added new benchmarks' # 跳过add命令直接提交

$ git pull # 同步远程仓库最新代码到本地

$ git push origin [分支名] # 推送代码到远程分支（加 -u 参数是同时关联远程与本地分支，一般新分支第一次推送时使用）
```
### 1.3 分支相关操作
```
$ git branch # 查看分支

$ git branch -v # 查看各分支最后一个提交对象

$ git branch --merged # 查看已经merge过的分支

$ git branch --no-merged # 尚未merge的分支

$ git branch -d testing # 删除掉分支(如果还没有merge,会出现错误,-D可以强制删除)

$ git branch -a # 查看所有分支（包括远程服务器）

$ git checkout iss53 # 切换到iss53分支

$ git checkout -b iss53 # 新建分支并切换到iss53分支 = $ git branch iss53; git checkout iss53

$ git merge dev # 合并dev分支到当前分支上（合并前要先切换到合并到的目标分支上）
```
### 1.4 状态查看相关操作
```
$ git log  # 查看提交log

$ git log --pretty=format:"%h - %an, %ar : %s" # 用特性的format查看log

$ git log --graph # 用图表的形式显示git的合并历史

$ git log --pretty=oneline # 简洁方式显示提交历史

$ git reflog # 查看所有提交历史，即使是已经被回退了的提交

$ git status # 检查文件当前的状态

$ git diff    #查看本地当前修改

$ git diff --cached # 若要看已经暂存起来的文件和上次提交时的快照之间的差异

$ git show commitid. 查看某次提交的修改
```
### 1.5 代码回退相关操作
```
$ git reset --hard HEAD^ # 将已经commit的代码回退到指定版本（HEAD^^是上上个版本，HEAD～3是上上上个版本，以此类推）

$ git reset --hard commit_id # 将版本回退到指定commitid，注意这个也可以向前进找到已经被回退之前的代码

$ git checkout -- readme.txt # 将工作区的修改回退，也就是将还未add的内容回退

$ git reset HEAD readme.txt # 将暂存区的修改回退，也就是将已经add还没有commit的内容回退（工作区修改依然存在，只是不在暂存区了）

$ git rm test.txt # 从版本库中删除文件

$ git rm --cached log.log # 从git仓库中删除不小心追踪的文件（用于gitignore之前追踪的文件） 
```
# 2、常见场景下git操作流程
### 2.1 远程仓库回退
本场景主要是本地代码出现问题，然后本地仓库回退了代码，同时也需要回退远程仓库的代码。

流程：

* 先回退本地代码，具体回退操作可以参见1.5命令
* 然后强制push到远程分支上去： git push origin mybranch -f

### 2.2 删除本地以及远程仓库分支
本场景主要是自己创建了临时分支，提交代码后临时分支不再需要，可以删除清理。

流程：

* 本地先切换到非要删除分支上
* 然后删除本地指定分支：git branch -D mybranch
* 再之后删除远程分支：git push origin --delete mybranch

### 2.3 提交review后重新修改
本场景主要是已经commit并push了一次代码，发起review后还需要重新修改代码，但是又不想重新再开一个commit（个人分支仅仅push但还没有发起merge request 或者 已经发起了merge request均适用）。

流程：

* 修改本地代码
* add代码：git add --all/./修改的文件
* 重新commit：git commit --amend
* push代码：git push origin mybranch -f

### 2.4 解决merge冲突
本场景是自己分支新push的代码发起了merge request，但是在merge时发现跟master分支已有代码有冲突，这时需要解决冲突重新merge。

流程：

* 首先本地切换到自己的分支：git checkout mybranch
* 然后fetch一下最新代码：git fetch
* 然后将自己分支rebase一下：git rebase -i origin/master(要rebase到的远程分支)
* 然后手动解决冲突（rebase后会提示哪些文件有冲突，有冲突的文件里面会有冲突的代码块，直接修改成最终的正确代码即可）
* add冲突文件：git add 冲突文件
* 继续rebase：git rebase --continue（这里如果还有冲突会继续提示冲突，继续修改然后继续add、rebase --continue直到没有冲突为止）
* push代码：git push origin mybranch -f
