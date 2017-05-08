---
layout: post
title: "HEXO使用总结"
date: 2016-07-25 16:15:06 
description: "HEXO配置，HEXO+Github，搭建自己的博客"
categories: 
    - 博客
tags:
    - hexo
    - github
---

本文将总结性的介绍如何建立自己的github.io博客，后续会持续补充，进阶。感谢[baixin](https://baixin.io)提供的参考文章。

<!--more-->

技术选型为github+hexo+idea，首先最简单的阐述下这个东西都干嘛的

## 一、技术选型

### 1)github
*   免费空间挂载网站。(这个好理解，提供username.github.io直接可以访问)
*   发布网站。(将hexo生成的网站推送到username.github.io上)
*   版本控制。(将整个网站的源文件推送到另一个repo上，便于网站开发环境迁移，多机工作等)


### 2)hexo
*   生成博客框架页面，可以通过md解析文章渲染页面，然后发布到github上，多种风格支持，扩展插件丰富。
*   顺便对比下`hexo`和`jekyll`
    *   hexo 基于`nodejs`，实施起来简便。
    *   jekyll 基于`ruby`，实施起来折腾。 但是支持在线编辑
      
              
### 3)idea
*   工作环境，写作环境。(git插件支持强大，md支持预览)
 
 
## 二、具体部署方案
    
### 1)当然是先申请github,在此不再赘述。需要注意的是要申请两个
*   username.github.io只需要申请下来存放页面，发布只需要配置好_config.yml文件的deploy属性，通过`hexo d`发布
*   username.github.io.hexo 整个hexo完整项目,方便迁移，多机协作。 
        
### 2)在项目中安装hexo，并按照其规则把主题样式放在themes里面(我们选择的是next)，并配置好_config.yml

### 3)扩展网站功能，安装扩展插件比如评论、百度统计。

### 4)编写md文件，可以写博文啦。

## 三、废话少说上代码，注意以下针对已经构建好的项目，再次迁移。

>   克隆项目

```bash
$ git clone https://github.com/psiitoy/psiitoy.github.io.hexo.git
cd psiitoy.github.io.hexo
```


>   安装hexo

```bash
$ npm install hexo --save


```


>   初始化hexo，添加git插件支持

```bash
$ hexo init --no-clone
$ npm install hexo-deployer-git --save

```


>>  `hexo init`构建前项目结构_config.yml  package.json  scaffolds  source  themes
>>  `hexo init`构建后项目结构_config.yml  package.json  scaffolds  source  themes  node_modules  public   db.json
>>  多了三个文件`db.json` `public` `node_modules`,同时git信息都没了...T.T,不慌继续


>   hexo的文件结构

```bash
_config.yml     #主站的配置文件
node_modules  
package.json    #应用程序的信息
scaffolds       #模版文件夹，新建文章时会根据 scaffold里对应md生成文件
source          #存放用户资源，除 _posts 文件夹之外， _ 开头的文件/和隐藏的文件将会被忽略
themes          #主题文件夹

```


>   初始化git，设置'git push'只提交当前分支，禁用lf自动转换，禁用中文文件名转换

```bash
$ git init
$ git config --global push.default simple
$ git config --global core.autocrlf false
$ git config --global core.quotepath false

```


>   追踪项目

```bash
$ git remote add origin git@github.com:psiitoy/psiitoy.github.io.hexo.git

```


>   纳入版本控制

```bash
$ git add .
$ git commit -m 'first commit'

```


>   更新分支

```bash
$ git pull origin master

```


>   然后运行idea rebase并且解决冲突(git能力有限，交给idea搞了)

>>  revert冲突文件
>>  commit解决冲突
>>  rebase到远程分支


>   ok了(idea暂时只能commit不能push待解决)，一切git操作交给命令行


## 四、hexo备注 

一些常用命令：

	hexo new "postName" #新建文章
	hexo new page "pageName" #新建页面
	hexo generate #生成静态页面至public目录
	hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
	hexo deploy #将.deploy目录部署到GitHub
	hexo help  # 查看帮助
	hexo version  #查看Hexo的版本

### 问题解决

#### github.io js 404无法正常显示
- 最近github page更新了，GitHub Pages 过滤掉了 source/vendors 目录的访问，所以next主题下的source下的vendors目录不能够被访问到，所以就出现了本地hexo s能够正常访问，但是deploy到github就是一片空白，按f12，可以看到大量来自source/vendors的css和js提示404

> 方法一（来自github next主题issue）:
首先修改source/vendors为source/lib，然后修改_config.yml， 将 _internal: vendors修改为_internal:lib 然后修改next底下所有引用source/vendors路径为source/lib。这些地方可以通过文件查找找出来。主要集中在这几个文件中。1. Hexo\themes\next.bowerrc 2. Hexo\themes\next.gitignore 3. Hexo\themes\next.javascript_ignore 4. Hexo\themes\next\bower.json 。修改完毕后，刷新重新g一遍就ok啦。

> 方法二:更新next主题，不过听过最新的next主题对第三方例如多说删除了，具体不清楚，不敢亲易尝试，毕竟更新一次主题引来的问题太多，很多配置可能都要改，代价太高，所以推荐第一种方法

	
## 五、Markdown语法参考链接
[很实用的例子](https://www.zybuluo.com/mdeditor)


## 六、Hexo参考链接
[通过Hexo在GitHub搭站全记录](https://anonymalias.github.io/2016/01/14/hexo-construct-homepage/)
[HEXO搭建个人博客](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)
[搭建 Hexo 博客--增强篇](http://www.jianshu.com/p/2640561e96f8)
[next主题的配置和优化](http://blog.csdn.net/willxue123/article/details/50994852)
[HEXO增加标签页、分类页](https://github.com/iissnan/hexo-theme-next/wiki/)
[HEXO next主题增加畅言评论系统](http://www.zhaiqianfeng.com/2017/03/changyan-for-hexo-next-theme.html)
