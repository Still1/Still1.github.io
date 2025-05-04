---
title: 解决Maven项目报错java.lang.NoClassDefFoundError的问题
tags: [Maven, 问题解决]
---

## 1

### 问题原因

Maven下载的jar包有问题不完整

### 解决方法

删掉本地仓库相关的包，重新更新

## 2

### 问题原因

依赖的包是间接依赖过来的，但Maven配置中开了optional，实际上没有依赖相应的包

### 解决方法

关掉optional，或者显式直接依赖开了optional的包