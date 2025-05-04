---
title: 解决Hibernate数据无法save，且后台无报错的问题
tags: [Hibernate, 问题解决]
---

## 问题原因

这是由于没有事务造成的

## 解决方法

方法加上注解
```java
@Transactional(readOnly = false, rollbackFor = Exception.class)
```

或者可以利用hibernate的session开始事务