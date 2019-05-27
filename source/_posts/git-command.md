layout: post
title: Git 常 用 命 令
author: Pyker
categories: CI/CD
img: /images/bar/gitcommand.jpg
tags:
  - git
top: false
cover: false
date: 2018-11-30 17:30:00
---
以下总结了日常常用的git指令方便读者学习和测试，如果想了解更多git指令相关的命令和参数，请参考[Git指令指南](https://git-scm.com/docs)
```bash
git init                                                  # 初始化本地git仓库（创建新仓库）
git config --global user.name "xxx"                       # 配置用户名
git config --global user.email "xxx@xxx.com"              # 配置邮件
git config --global color.ui true                         # git status等命令自动着色
git config --global color.status auto
git config --global color.diff auto
git config --global color.branch auto
git config --global color.interactive auto
git clone git+ssh://git@192.168.53.168/VT.git             # clone远程仓库
git status                                                # 查看当前版本状态（是否修改）
git add xyz                                               # 添加xyz文件至暂存区
git add .                                                 # 增加当前子目录下所有更改过的文件至暂存区
git commit -m 'xxx'                                       # 提交
git commit --amend -m 'xxx'                               # 合并上一次提交（用于反复修改）
git commit -am 'xxx'                                      # 将add和commit合为一步
git rm xxx                                                # 删除暂存区中的文件
git rm -r *                                               # 递归删除
git log                                                   # 显示提交日志
git log -1                                                # 显示最近1次commit的日志 -n为最近n次
git log -5 --oneline									  # 极简显示最近5次的commit日志
git log --stat                                            # 显示提交日志及相关变动文件
git log --graph --pretty=oneline --abbrev-commit          # 以图形表示分支合并历史
git show dfb02e6e4f2f7b573337763e5c0013802e392818         # 显示某个提交的详细内容
git show dfb02                                            # 可只用commitid的前几位
git show HEAD                                             # 显示HEAD提交日志
git show HEAD^                                            # 显示HEAD的父的提交日志
git show HEAD~10                                          # 显示前面第10次的提交日志
git show -s --pretty=raw 2be7fcb476
git tag                                                   # 显示已存在的tag
git tag -a v2.0 -m 'xxx'                                  # 增加v2.0的tag
git show v2.0                                             # 显示v2.0的日志及详细内容
git log v2.0                                              # 显示v2.0的日志
git diff                                                  # 显示所有未添加至暂存区的变更
git diff --cached                                         # 显示所有已添加index但还未commit的变更
git diff HEAD^                                            # 比较与上一个版本的差异
git diff HEAD -- ./lib                                    # 比较与HEAD版本lib目录的差异
git diff origin/master..master                            # 比较远程分支master与本地master分支的区别
git diff origin/master..master --stat                     # 只显示差异的文件，不显示具体内容
git remote add origin git+ssh://git@192.168.53.168/VT.git # 增加远程地址定义（用于push/pull/fetch）
git branch                                                # 显示本地分支
git branch --contains 50089                               # 显示包含提交50089的分支
git branch -a                                             # 显示所有分支
git branch -r                                             # 显示所有原创分支，包括本地和远程跟踪分支
git branch --merged                                       # 显示所有已合并到当前分支的分支
git branch --no-merged                                    # 显示所有未合并到当前分支的分支
git branch -m master master_copy                          # 本地分支重命名
git checkout -b master_copy                               # 从当前分支创建新分支master_copy并检出
git checkout -b dev origin/dev                            # 拉取远程dev分支到本地dev分支
git checkout features/performance                         # 检出已存在的features/performance分支
git checkout --track hotfix/BJP93                         # 检出远程分支hotfix/BJP93并创建本地跟踪分支
git checkout v2.0                                         # 检出版本v2.0
git checkout -b devel origin/develop                      # 从远程分支develop创建新本地分支devel并检出
git checkout -- README                                    # 将README文件回退到更新前(用于修改错误回退)
git merge --no-ff origin/master                           # 合并远程master分支至当前分支
git cherry-pick ff44785404a8e                             # 合并提交ff44785404a8e的修改
git push origin master                                    # 将当前master分支push到远程master分支
git push origin :hotfixes/BJVEP933                        # 删除远程仓库的hotfixes/BJVEP933分支
git push --tags                                           # 把所有tag推送到远程仓库
git push --set-upstream origin dev                        # 推送本地当前分支到远程dev分支
git fetch                                                 # 获取所有远程分支（不更新本地分支，需merge）
git fetch --prune                                         # 获取所有原创分支并清除服务器上已删掉的分支
git fetch origin dev:tmp                                  # 拉取远程dev分支到本地tmp分支
git pull origin master                                    # 获取远程分支master并merge到当前分支
git mv README README2                                     # 重命名文件README为README2
git reset --hard 2f3c                                     # 将当前版本重置为2f3c（常用于merge回退）
git rebase
git branch -d hotfix                                      # 删除分支hotfix (该分支已合并到其他分支)
git branch -D hotfix/BJP93                                # 强制删除分支hotfix/BJP93
git ls-files                                              # 列出git 暂存区包含的文件
git show-branch                                           # 图示当前分支历史
git show-branch --all                                     # 图示所有分支历史
git whatchanged                                           # 显示提交历史对应的文件修改
git revert dfb02e6e4                                      # 撤销提交dfb02e6e4
git ls-tree HEAD                                          # 内部命令：显示某个git对象
git rev-parse v2.0                                        # 内部命令：显示某个ref对于的SHA1 HASH
git reflog                                                # 显示历史所有提交，包括孤立节点
git show HEAD@{5}
git show master@{yesterday}                               # 显示master分支昨天的状态
git log --pretty=format:'%h %s' --graph                   # 图示提交日志
git stash                                                 # 暂存当前修改，将所有至为HEAD状态
git stash list                                            # 查看所有暂存
git stash show -p stash@{0}                               # 参考第一次暂存
git stash apply stash@{0}                                 # 应用第一次暂存
git stash drop                                            # 删除stash记录
git stash pop                                             # 等价于 git stash apply和git stash drop
git grep "delete from"                                    # 文件中搜索文本“delete from”
git grep -e '#define' --and -e SORT_DIRENT
git gc
git fsck
```
### git fetch 和git pull 的差别
* git fetch 相当于是从远程获取最新到本地，不会自动merge，如下指令：
```bash
git fetch                                 # 将远程仓库的当前分支下载到本地当前branch中
git diff origin/master                    # 比较本地的master分支和origin/master分支的差别
git diff HEAD FETCH_HEAD                  # 和上条命令一样效果
git merge origin/master                   # 进行合并
```
也可以用以下指令：
```bash
git fetch origin master:tmp        # 从远程仓库master分支获取最新，在本地建立tmp分支
git diff tmp                       # 将当前分支和tmp進行對比
git merge tmp                      # 合并tmp分支到当前分支
```
* git pull：相当于是从远程获取最新版本并merge到本地
```bash
git pull origin master
```
git pull 相当于从远程获取最新版本并merge到本地
在实际使用中，git fetch更安全一些