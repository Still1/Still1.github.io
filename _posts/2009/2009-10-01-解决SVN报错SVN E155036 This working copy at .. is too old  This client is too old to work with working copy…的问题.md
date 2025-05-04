---
title: 解决SVN报错SVN E155036 This working copy at .. is too old  This client is too old to work with working copy…的问题
tags: [SVN, 问题解决]
---

## 问题描述

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405065725.png)

## 问题原因

SVN客户端版本与SVN的working copy格式版本不一致

## 解决方法

working copy too old可用svn upgrade命令行
client too old 升级客户端

