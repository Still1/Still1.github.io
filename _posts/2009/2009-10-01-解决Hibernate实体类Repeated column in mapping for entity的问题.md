---
title: 解决Hibernate实体类Repeated column in mapping for entity的问题
tags: [Hibernate, 问题解决]
---

## 问题描述

```
Repeated column in mapping for entity: XXpojo column: xx (should be mapped with insert="false" update="false")
```

## 问题原因

这是由于同一个数据库列映射到多个成员变量上

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080710.png)

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080716.png)

## 解决方法

取消重复，或加上在注解配置上`insert="false" update="false"`