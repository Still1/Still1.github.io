---
title: 解决MySQL主从架构 UUID重复报错的问题
tags: [MySQL, 问题解决]
---

## 问题描述

```
Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
```

## 解决方法

修改data文件夹下的auto.cnf配置文件，换一个uuid