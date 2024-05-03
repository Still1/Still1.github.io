---
title: 问题解决（Hibernate）
tags: [Hibernate, 问题解决]
---

## Hibernate数据无法save，且后台无报错

这是由于没有事务造成的，方法加上注解
```java
@Transactional(readOnly = false, rollbackFor = Exception.class)
```

或者可以利用hibernate的session开始事务

## Hibernate找不到session

报错

2016-06-06 15:43:24 2016-06-06 15:43:24,525 - ERROR LogInit( 55 ) -*org.hibernate.HibernateException*: No Hibernate Session bound to thread, and configuration does not allow creation of non-transactional one here

原因是没有加入事务

将`@Transactional`加到方法或类，例如可以加到DAO的类，也可以加到调用DAO的方法上（service的方法）

加到service类上不会报错，但数据库操作不会提交，原因不明，可能与代理形式有关

## Hibernate实体类Repeated column in mapping for entity

报错信息：Repeated column in mapping for entity: XXpojo column: xx (should be mapped with insert="false" update="false")

原因：这是由于同一个数据库列映射到多个成员变量上

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080710.png)

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080716.png)

解决方法：取消重复，或加上在注解配置上`insert="false" update="false"`