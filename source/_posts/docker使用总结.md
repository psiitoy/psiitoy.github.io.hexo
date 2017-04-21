---
layout: post
title: "docker使用总结"
date: 2016-07-25 21:15:06 
description: "docker使用总结"
categories: 
    - docker
tags:
    - docker
---

docker使用总结

<!--more-->

学习 安装 http://www.widuu.com/docker/installation/ubuntu.html

下载
sudo apt install docker.io
赋权
sudo usermod -aG docker wangdi
查看是否不用sudo
docker images
查镜像从docker hub 
docker search ubuntu
下载镜像
docker pull ubuntu //这个版本啥都没有无法弄
docker pull rastasheep/ubuntu-sshd
安装容器
$ sudo docker run -d -P --name test_sshd rastasheep/ubuntu-sshd:14.04
$ sudo docker port test_sshd 22
  0.0.0.0:49154

$ ssh root@localhost -p 49154
# The password is `root`
root@test_sshd $

进入容器
docker start -ai f3c234a93088