---
title: 解决JDBCTemplate queryForList查询出来的某些数字自动转换为true和false的问题
tags: [Spring Data, 问题解决]
---

## 问题原因

MySQL数据库的字段数据类型为TINYINT

## 解决方法

转换为SMALLINT