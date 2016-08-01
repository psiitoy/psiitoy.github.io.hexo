---
layout: post
title: "git总结"
date: 2016-08-01 10:45:06 
description: "git总结"
categories: 
    - git
tags:
    - git
---

文章简要记录部分常用git命令。本文感谢[廖雪峰](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)提供的详细git教程。

<!--more-->

## 自报家门

> 注意`git config`命令的`--global`参数，是机器维度上的全局配置，可以对某个仓库指定不同的用户名和email地址。

```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

## 查看版本信息

```bash
git status
```

## 查看提交日志

> 加上`--pretty=oneline`使每个提交显示在一行
> 查看提交历史来决定要回退到那个版本

```bash
git log --pretty=oneline
```

## 查看操作日志

> 用来查看每一次命令的记录
> 可以回滚之后再恢复,“回到未来”

```bash
git reflog
```

## 回滚版本

> 回退到上一个`commit`的版本

```bash
git reset --hard HEAD^
```

> 回退到某一个`commit`的版本

```bash
git reset --hard commit_id
```

## 撤销更改

> 撤销单个文件的更改，使改文件返回add之前的状态 

```bash
git checkout -- file
```

## 关联远程仓库

> 添加后，远程库的名字就是`origin`，这是git默认的叫法，可以改成别的。但是这一看就是远程库。
> 同时，放心你们推不上去，因为你的`ssh key`不在我的账户列表中

```bash
git remote add origin git@github.com:psiitoy/art.git
```

## 把本地库的所有内容推送到远程库上

> 由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
> `-u`是关联的意思

```bash
git push -u origin master
```

> 以后本地做了提交想推送到远程就是

```bash
git push origin master
```

## 创建分支

> 创建`<name>`分支，然后切换到`<name>`分支

```bash
git checkout -b <name>
```

> 等同于

```bash
git branch <name>
git checkout <name>
```

> 查看当前分支

```bash
git branch
```

> 切换分支

```bash
git checkout <name>
```

> 合并分支

```bash
git merge <name>
```

> 删除分支

```bash
git branch -d <name>
```