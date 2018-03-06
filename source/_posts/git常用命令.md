---
title: git常用命令
date: 2017-09-19 10:50:53
tags:
---

### Git global setup

``` bash
$ git config --global user.name "Administrator"
$ git config --global user.email "admin@example.com"
```

### Git project setup

``` bash
$ git config user.name "Listing"
$ git config user.email "admin@example.com"
```

### Git bash

``` bash
$ git init  # 创建版本库
$ git add . # 把文件添加到仓库
$ git commit -m "" # 把文件提交到仓库  -m 提交说明
$ git remote add origin remote_git_address # 关联远程仓库
$ git push -u origin master # 第一次把本地库的所有内容推送到远程库上
$ git push origin master # 推送内容
$ git remote set-url origin remote_git_address # 更新仓库地址
$ git status # 查看仓库当前状态
$ git diff # 查看difference  修改内容
$ git log --pretty=oneline # 显示从最近到最远的提交日志 --pretty=oneline 参数 输出信息 id 和说明
$ git reflog # 查看命令历史
$ git reset --hard commit_id # 回退到某一版本
$ git checkout — file # 撤销修改
$ git branch -r # 查看当前分支
$ git branch <name> # 创建分支
$ git checkout <name> # 切换分支
$ git checkout -b <name> # 创建+切换分支
$ git merge <name> # 合并某分支到当前分支
$ git branch -d <name> # 删除分支
$ git checkout -- <file> # 取消对文件的修改。还原到最近的版本，废弃本地修改
$ git reset 057d # 回退到某个版本
$ git reset HEAD <file> # 取消已经暂存的文件
$ git reset HEAD^ # 回退所有内容到上一个版本
$ git reset HEAD^ <file> # 回退文件到上一个版本
$ git reset –hard origin/master # 将本地的状态回退到和远程的一样
```
