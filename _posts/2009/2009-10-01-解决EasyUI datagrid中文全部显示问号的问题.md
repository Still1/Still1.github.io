---
title: 解决EasyUI datagrid中文全部显示问号的问题
tags: [EasyUI, 问题解决]
---

## 解决方法

servlet的response调用

```java
response.setContentType("text/html; charset=UTF-8");
```
