---
title: Git常用命令
layout: post
date: 2016-07-13 00:00:00
tags: [Reference]
feature-img: "https://images.pexels.com/photos/450035/pexels-photo-450035.jpeg?cs=srgb&dl=business-computer-connection-450035.jpg&fm=jpg"
---


由于Git命令不是太熟悉，常规的就那么几条，为了方便自己的使用查找，遂整理出来。

<!--more-->

## 初始化

```bash
git init  # 初始化版本库
git init --bare # 创建裸仓库，无工作区
```

## 常规使用

```bash
git add   # 添加文件到版本库
git commit# 确认提交

git status #查看工作区状态
git diff   # 查看修改内容

git log    #查看日志
git log --pretty=oneline # 以一行的形式查看

# HEAD表示当前版本，HEAD^表示上一版本，以此类推，回退到上一版本命令为
git reset --hard HEAD^

# 查看当前版本下，过去和未来的版本记录
git reflog

# 撤销修改  注意区分工作区与暂存区，实际操作时，则查看提示
git checkout -- file  # -- 表示当前分支; 丢弃工作区修改
git reset HEAD file   # 丢弃暂存区修改

git rm   # 删除文件
git mv   # 移动或者重命名文件or目录

# 添加远程仓库
git remote add origin git@server-name:path/repo-name.git
git push -u origin master # 首次推送

# 克隆仓库
git clone git@github.com:name/repo-name

git branch -D <name>  # 强行删除未合并的分支
git push origin <name> # 推送指定分支到远程
```

## 分支管理

```shell
# 分支管理
git checkout -b dev #创建并切换到分支dev
git branch dev ; git checkout dev
git branch   # 查看当前分支
git branch -d <name> # 删除分支
git merge <name>   # 合并某分支到当前分支
git log --graph  # 查看分支合并图
git merge --no-ff  # 以普通模式合并，不删除分支
```

## 保存现场

当现在进行手头上的工作时，又需要创建新的分支修BUG，当前任务又不允许你提交的前提下，可进行保留现场的操作。

```shell
git stash # 保存现场
git stash list  # 查看工作现场
git stash apply # 恢复现场 
git stash drop  # 删除现场
git stash pop   # 恢复的同时删除现场
```

## 标签管理

```shell
git tag -a <version> -m "content"  # 打一个新标签
git tag <name> commit id  # 指定的提交打标签
git tag  # 查看标签
git show <tagname> #  查看标签详细内容
git tag -d <tagname> # 删除标签
git push origin <tagname> # 推送某个标签
git push origin --tags    # 一次性推送所有本地标签
```

## 自定义与别名

```shell
# 自定义git
git config --global color.ui true
.gitignore  # 定义需要忽略的文件
git add -f <file name> # 强制添加到暂存区

# 定义别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.last 'log -l' # 最后提交
# 这个是大招
git  config --global alias.lg "log --color --graph \
    --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset \
    %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

.git/config  # 针对仓库的配置文件
~/.gitconfig  # 针对用户的配置文件
```

## 分支管理原则

1. 以 master 和 dev 分支为主，其他均为临时性的分支
2. 临时性分支又分3种
   1. 功能「feature」分支
   2. 预发布「release」分支
   3. 修补bug「fixbug」分支
3. 每次合并到主分支时，记得打标签「tag」
4. 每次创建临时分支时，要确认清楚需要以哪个分支为基础创建
5. 只保留主分支和开发分支到远程仓库，其他分支仅保留在本地


## 参考链接


> [廖雪峰GIT教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
>
> [GIT 简易指南](http://www.bootcss.com/p/git-guide/)
>
> [网友小结的命令](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
