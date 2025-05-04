---
title: 解决Tomcat或Jetty中间件不扫描WebApplicationInitializer的问题
tags: [Tomcat, Jetty, 问题解决]
---

## 解决方法

添加3.1版本的web.xml

```xml
<web-app  xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee  http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1">
</web-app>
```