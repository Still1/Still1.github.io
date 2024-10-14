---
title: MyBatis-Plus FieldStrategy配置
tags: [MyBatis-Plus]
---

## FieldStrategy

通过配置FieldStrategy，可以控制在数据库操作时，对应的字段是否参与。

## FieldStrategy取值

* `IGNORED` 任何值都处理，包含null
* `NOT_NULL` null值忽略不处理
* `NOT_EMPTY` null值和空字符串忽略不处理
* `NEVER` 任何值都不处理
* `DEFAULT` 按全局配置

## 单字段配置

可分别配置update、insert、where场景下的策略
```java
@TableField(updateStrategy = FieldStrategy.IGNORED, insertStrategy = FieldStrategy.IGNORED, whereStrategy = FieldStrategy.IGNORED)
```

## 全局配置

```yml
mybatis-plus:  
  global-config:  
    db-config:  
      update-strategy: ignored  
      insert-strategy: ignored
```