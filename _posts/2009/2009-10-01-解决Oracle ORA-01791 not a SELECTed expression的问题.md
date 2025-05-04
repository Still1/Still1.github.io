---
title: 解决Oracle ORA-01791 not a SELECTed expression的问题
tags: [Oracle, 问题解决]
---

## 问题描述

执行语句

```sql
SELECT distinct pd_crea_co_code FROM pd_project_jx_de LEFT JOIN t_unit ON t_unit.unitno = pd_project_jx_de.pd_crea_co_code LEFT JOIN t_department ON t_department.code = pd_project_jx_de.xmks order by unitname asc;
```

报错
ORA-01791: not a SELECTed expression 
01791.00000 -  "not a SELECTed expression"

## 问题原因

当select子句包含distinct时，排序的字段必须在select子句里面存在