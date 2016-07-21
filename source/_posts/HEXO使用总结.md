---
layout: post
title: "HEXO使用总结"
date: 2015-08-25 21:15:06 
description: "HEXO配置，HEXO+Github，搭建自己的博客"
category: 博客
---

本文将总结性的介绍如何建立自己的github.io博客，后续会持续补充，进阶。感谢[baixin](https://baixin.io)提供的参考文章。


<!--more-->

技术选型为github+hexo+idea，首先最简单的阐述下这个东西都干嘛的
## 一、技术选型

* #### 1)github
    *   免费空间挂载网站。(这个好理解，提供username.github.io直接可以访问)
    *   发布网站。(将hexo生成的网站推送到username.github.io上)
    *   版本控制。(将整个网站的源文件推送到另一个repo上，便于网站开发环境迁移，多机工作等)

* #### 2)hexo
    *   生成博客框架页面，同样产品jekyll相对来说更复杂，实验过后还是选择hexo。(可以通过md解析文章渲染页面，多种风格支持，扩展插件丰富)。
      
* #### 3)idea
    *   工作环境，写作环境。(git插件支持强大，md支持预览)
 
## 二、具体部署方案
### 主要针对对以上技术有大概了解的朋友，作为一个最佳实践来记录。以后再上图。
    
* #### 1)当然是先申请github,在此不再赘述。需要注意的是要申请两个，一个是username.github.io存放页面，另一个是username.github.io.hexo存放hexo项目文件方便迁移。
    *   username.github.io只需要申请，发布只需要配置好_config.yml文件的deploy属性，通过hexo d发布
    *   username.github.io.hexo分为初次构建(从头搞起来hexo)和二次构建(已经有hexo框架只是重新下项目)
        *   初次  构建hexo项目，删除themes里面的git信息，删除项目根目录下的.depoly_git文件夹，修改.gitignore文件
        *   二次
        
* #### 2)


hexo 构建前(项目文件)
_config.yml  node_modules  public     source
db.json      package.json  scaffolds  themes

hexo 构建后(github上存放的文件)
_config.yml  package.json  scaffolds  source  themes

初次
npm install hexo --save
hexo init 之后出现了db.json public node_modules 同时 git信息都没了...T.T

