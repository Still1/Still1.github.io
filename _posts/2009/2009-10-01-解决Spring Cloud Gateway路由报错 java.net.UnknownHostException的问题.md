---
title: 解决Spring Cloud Gateway路由报错 java.net.UnknownHostException的问题
tags: [Spring Cloud, 问题解决]
---

## 问题描述

报错信息

```
java.net.UnknownHostException: renren-fast
```

## 问题原因

路由配置uri为http://renren-fast

## 解决方法

路由配置uri为lb://renren-fast