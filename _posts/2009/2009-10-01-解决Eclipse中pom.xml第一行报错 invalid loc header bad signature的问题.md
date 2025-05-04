---
title: 解决Eclipse中pom.xml第一行报错 invalid loc header bad signature的问题
tags: [Eclipse, 问题解决]
---

## 问题原因

Maven一些依赖包下载不正确

## 解决方法

运行maven test等命令或用Jetty启动项目，可以看到报错信息里面显示的有问题的包路径，删掉重新update下载即可