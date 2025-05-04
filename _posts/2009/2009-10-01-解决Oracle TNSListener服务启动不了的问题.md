---
title: 解决Oracle TNSListener服务启动不了的问题
tags: [Oracle, 问题解决]
---

## 问题描述

启动TNSListener，启动过程中弹出警告框，大意是启动过程中又停止了

## 解决方法

将Oracle配置文件 listener.ora 和 tnsnames.ora里面，与访问不到的IP有关的配置删掉
如果不行，再尝试将localhost改为计算机名