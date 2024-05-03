---
title: SQL中delete和truncate
tags: [SQL]
---

## 作用

都可以用于删除一张数据表的全部数据

```sql
-- 删除表a的全部数据
delete from a;
truncate table a;
```

## 区别

1. `delete`属于DML，而`truncate`属于DDL
2. `truncate`只能用于表，而`delete`可以用于表、视图等
3. `delete`可以加`where`子句限制条件，`truncate`不能
4. `delete`支持事务，`truncate`不支持
5. `truncate`删除效率比`delete`高

## 总结

一般来说，只有删除整张表的数据，表的数据量大，且无需考虑撤回的情况下，可以考虑使用`truncate`以提升性能