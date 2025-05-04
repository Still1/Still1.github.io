---
title: 解决jQuery AJAX请求，数据中的数组无法正常映射到Java对象中的问题
tags: [jQuery, 问题解决]
---

## 问题描述

```
java.lang.NumberFormatException:  For  input  string:  ""
```

### 解决方法

加上traditional : true
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080257.png)
加上后请求参数正常：
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405080322.png)

如不加上，请求参数会变成数组

https://stackoverflow.com/questions/25134795/pass-array-into-spring-mvc-by-ajax
