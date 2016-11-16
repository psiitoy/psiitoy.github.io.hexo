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
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

> 让`git`显示颜色，让命令输出看起来更醒目

```bash
$ git config --global color.ui true
```

> 防止中文文件名或者路径转义成xxx

```bash
$ git config --globalcore.quotepath false
```

> 回车符号自动转换
>> `true`提交时转换为LF，检出时转换为CRLF
>> `input`提交时转换为LF，检出时不转换
>> `false`提交检出均不转换

```bash
$ git config --global core.autocrlf false
```

> 拒绝提交包含混合换行符的文件
>> `true`拒绝
>> `false`允许
>> `warn`警告

```bash
$ git config --global core.safecrlf true   
```

> 简易推送，意味着如果没有指定分支，推送当前分支

```bash
$ git config --global push.default simple
```

## 查看版本信息

```bash
$ git status
```

## 查看提交日志

> 加上`--pretty=oneline`使每个提交显示在一行
> 查看提交历史来决定要回退到那个版本

```bash
$ git log --pretty=oneline
```

## 查看操作日志

> 用来查看每一次命令的记录
> 可以回滚之后再恢复,“回到未来”

```bash
$ git reflog
```

## 回滚版本

> 回退到上一个`commit`的版本

```bash
$ git reset --hard HEAD^
```

> 回退到某一个`commit`的版本

```bash
$ git reset --hard commit_id
```

> 本地master分支reset之后push的时候要加上-f参数
```
$ git push -f origin master 
```

## 撤销更改

> 撤销单个文件的更改，使改文件返回add之前的状态 

```bash
$ git checkout -- file
```

## 关联远程仓库

> 添加后，远程库的名字就是`origin`，这是git默认的叫法，可以改成别的。但是这一看就是远程库。
> 同时，放心你们推不上去，因为你的`ssh key`不在我的账户列表中

```bash
$ git remote add origin git@github.com:psiitoy/art.git
```

## 把本地库的所有内容推送到远程库上

> 由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
> `-u`是关联的意思

```bash
$ git push -u origin master
```

> 以后本地做了提交想推送到远程就是

```bash
$ git push origin master
```

## 创建分支

> 创建`<name>`分支，然后切换到`<name>`分支

```bash
$ git checkout -b <name>
```

> 等同于

```bash
$ git branch <name>
$ git checkout <name>
```

> 查看当前分支

```bash
$ git branch
```

> 查看所有分支信息，版本号和描述

```bash
$ git branch -va
```

> 切换分支

```bash
$ git checkout <name>
```

> 合并分支

```bash
$ git merge <name>
```

> 删除分支 `branch -d` 如果是删除一个没被合并的分支要用 `branch -D`强行删除

```bash
$ git branch -d <name>
```

## 合并分支

> 合并流程，先`merge`，再解决冲突，然后`add`,`commit`

```bash
$ git merge <fromBranch>
```

> 合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

```bash
$ git merge --no--ff -m 'merge with no-ff' <fromBranch>
```

> 查看分支合并图

```bash
$ git log --graph
```

## 暂存分支

> 暂存

```bash
$ git stash

Saved working directory and index state WIP on dev: 6224937 add merge
HEAD is now at 6224937 add merge
```

> 查看暂存列表

```bash
$ git stash list

stash@{0}: WIP on dev: 6224937 add merge
```

> 恢复工作现场并且删除该stash使用`pop`，如果不删则使用`apply`

```bash
$ git stash pop
```

> 恢复某个stash

```bash
$ git stash list
$ git stash apply stash@{0}
```

## 多人协作

> 查看远程库信息

```bash
$ git remote
origin
```

> 查看所有远程库分支信息

```bash
$ git remote show origin
```

> 查看远程库更详细的信息，是否有权限抓取`fetch`和推送`push`

```bash
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```

> 推送远程分支

```bash
$ git push origin <branchName>
```

> `git pull`失败，原因是没有指定本地`dev`分支与远程`origin/dev`分支的链接，根据提示，设置`dev`和`origin/dev`的链接。

```bash
$ git branch --set-upstream dev origin/dev
Branch dev set up to track remote branch dev from origin.
```

> `git pull`和`git fetch`的区别
>> `git fetch <远程主机名> <分支名>`将某个远程主机的更新，全部取回本地。默认取回所有分支的更新。
>> `pull`相当于`fetch`+`merge`
>> 加上`-p`可以删除远程已经删除的分支

```bash
$ git pull -p
# 等同于下面的命令
$ git fetch --prune origin 
$ git fetch -p
```

> 多人协作的工作模式通常是这样：
  
>> 1.  首先，可以试图用git push origin branch-name推送自己的修改；
  
>> 2. 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
  
>> 3. 如果合并有冲突，则解决冲突，并在本地提交；
  
>> 4. 没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！
  
>> 5. 如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。
  
## 创建标签

> tag就是一个让人容易记住的有意义的名字，它跟某个`commit`绑在一起。打最新提交的commit或者指定`commit_id`，还可以加`-m`指定说明文字。

```bash
$ git tag v1.0

$ git tag v0.9 6224937

$ git tag -a v0.1 -m "version 0.1 released" 3628164
```

> 查看所有标签

```bash
$ git tag
v0.9
v1.0
```

> 查看标签信息

```bash
$ git show v0.9
```

> 删除标签

```bash
$ git tag -d v0.1
Deleted tag 'v0.1' (was e078af9)
```

> 推送某个标签到远程

```bash
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0
```

> 一次性推送全部尚未推送到远程的本地标签
```bash
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 554 bytes, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
 * [new tag]         v0.2 -> v0.2
 * [new tag]         v0.9 -> v0.9
```

> 如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除，然后，从远程删除。删除命令也是push，但是格式如下。

```bash
$ git tag -d v0.9
Deleted tag 'v0.9' (was 6224937)
$ git push origin :refs/tags/v0.9
To git@github.com:michaelliao/learngit.git
 - [deleted]         v0.9
```

## 配置别名

> 我们只需要敲一行命令，告诉Git，以后`st`就表示`status`

```bash
$ git config --global alias.st status
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch

```

> 甚至还有人丧心病狂地把`lg`配置成了：`git lg`

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 不小心提交错了不想纳入版本管理的文件(`gitignore`不起作用)

> 明明写好了规则,但问题不起作用,每次还是重复提交,无法忍受.其实这个文件里的规则对已经追踪的文件是没有效果的.所以我们需要使用rm命令清除一下相关的缓存内容.这样文件将以未追踪的形式出现.然后再重新添加提交一下,.gitignore文件里的规则就可以起作用了。

```bash
$ git rm -r --cached .
$ git add .
$ git commit -m 'update .gitignore'

```

## 运用shh -T -v git@github.com查看具体出错信息，再根据信息来调试

```bash
$ ssh -T psiitoy@git.com
Welcome to GitLab, psiitoy!

$ ssh -vT psiitoy@git.com
...

```

## 清理远端，已经合并过的分支

```bash
$ git br -r --merged | egrep -v "origin/master|origin/HEAD" | sed 's/origin\//:/g' | xargs git push origin
```

## git学习参考链接
[廖雪峰Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
[git简易指南](http://www.bootcss.com/p/git-guide/)