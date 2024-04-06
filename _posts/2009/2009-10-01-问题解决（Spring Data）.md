---
title: 问题解决（Spring Data）
tags: [软件开发, Spring, Spring Data]
---

## JDBCTemplate queryForList 查询出来的某些数字自动转换为true和false

原因是MySQL数据库的字段数据类型为TINYINT，转换为SMALLINT可解决