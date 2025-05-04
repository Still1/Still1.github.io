---
title: 解决Eclipse web module版本问题：Cannot change version of project facet Dynamic Web Module to 2.5.
tags: [Eclipse, 问题解决]
---

## 问题原因

`web.xml`中头配置版本不对应

## 解决方法

将原头配置
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    version="2.5">
```

修改为
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
```