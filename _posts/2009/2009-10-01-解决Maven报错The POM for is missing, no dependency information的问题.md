---
title: 解决Maven报错The POM for is missing, no dependency information的问题
tags: [Maven, 问题解决]
---

## 问题原因

运行`maven test`时，找不到自己写的另外的模块的依赖，出现错误

## 解决方法

依赖的模块运行install，安装到本地仓库

或者将项目组织为一个聚合项目，对整个聚合进行操作