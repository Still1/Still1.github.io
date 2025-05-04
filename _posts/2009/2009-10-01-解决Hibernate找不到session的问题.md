---
title: 解决Hibernate找不到session的问题
tags: [Hibernate, 问题解决]
---

## 问题描述

```
2016-06-06 15:43:24 2016-06-06 15:43:24,525 - ERROR LogInit( 55 ) -*org.hibernate.HibernateException*: No Hibernate Session bound to thread, and configuration does not allow creation of non-transactional one here
```

## 问题原因

没有加入事务

## 解决方法

将`@Transactional`加到方法或类，例如可以加到DAO的类，也可以加到调用DAO的方法上（service的方法）

加到service类上不会报错，但数据库操作不会提交，原因不明，可能与代理形式有关