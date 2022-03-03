---
title: MySQL group_concat函数例子
tags: 
  - MySQL
modify_date: 2017-10-20
---

## SQL例子

<!--more-->

```sql
SELECT t_book.id, t_book.chinese_name, t_book.name,
group_concat(t_author.chinese_name separator '/')
from t_book_author
left join t_author on t_author.id = t_book_author.author_id
left join t_book on t_book.id = t_book_author.book_id
group by t_book.id, t_book.chinese_name, t_book.name;
```

`group_concat`可看作为聚集函数，把目标字段的字符串连接起来，可指定分隔符