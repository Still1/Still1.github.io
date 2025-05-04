---
title: 解决Java使用HTTP代理报错java.io.IOException Unable to tunnel through proxy. Proxy returns  HTTP 1.1 407 Proxy Authentication Required的问题
tags: [Java, 问题解决]
---

## 问题描述

```
java.io.IOException: Unable to tunnel through proxy. Proxy returns "HTTP/1.1 407 Proxy Authentication Required"
```

## 问题原因

代理用户名密码使用`Authenticator.setDefault`设置的问题

## 解决方法

JDK 8u111版本后 默认将basic格式的鉴权禁止了
代码设置环境变量`System.setProperty("jdk.http.auth.tunneling.disabledSchemes", "");`打开basic格式
必须在`HTTPURLConnection`类初始化前

可以通过设置系统变量`jdk.http.auth.tunneling.disabledSchemes`，或设置JVM启动参数`-Djdk.http.auth.tunneling.disabledSchemes=""`，保证

也可以将`System.setProperty("jdk.http.auth.tunneling.disabledSchemes", "");`调用放到Spring Boot 启动类`main`方法中