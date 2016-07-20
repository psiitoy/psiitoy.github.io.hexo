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
    *  免费空间挂载网站。(这个好理解，提供username.github.io直接可以访问)
    *  发布网站。(将hexo生成的网站推送到username.github.io上)
    *  版本控制。(将整个网站的源文件推送到另一个repo上，便于网站开发环境迁移，多机工作等)

* #### 2)hexo
    *  生成博客框架页面，同样产品jekyll相对来说更复杂，实验过后还是选择hexo。(可以通过md解析文章渲染页面，多种风格支持，扩展插件丰富)。
      
* #### 3)idea
    *  工作环境，写作环境。(git插件支持强大，md支持预览)
 
## 二、具体部署方案

当然是先申请github,在此不再赘述。