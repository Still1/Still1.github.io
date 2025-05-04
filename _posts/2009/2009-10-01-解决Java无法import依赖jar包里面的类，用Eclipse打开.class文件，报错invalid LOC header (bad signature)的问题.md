---
title: 解决Java无法import依赖jar包里面的类，用Eclipse打开.class文件，报错invalid LOC header (bad signature)的问题
tags: [Java, 问题解决]
---

## 问题描述

编译提示
```
The import org.springframework.context.ApplicationContext cannot be resolved
```

## 问题原因

Maven下载的jar包文件有问题

## 解决方法

删除重新下载即可