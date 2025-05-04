---
title: 解决Eclipse校验XML时无法获取在线XSD文件报错的问题
tags: [Eclipse, 问题解决]
---

## 解决方法

>You could try adding a user specified catalog contribution to Eclipse. Under **Window->Preferences->XML->XML Catalog** select **User Specified Entries** and then the **Add** button.
>You can then add the details for the schema (you could copy the file locally in case of dropped connection). Eclipse will then have access to the schema during validation.

例子
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405074823.png)

https://stackoverflow.com/questions/901242/why-cant-eclipse-resolve-the-spring-dwr-schema