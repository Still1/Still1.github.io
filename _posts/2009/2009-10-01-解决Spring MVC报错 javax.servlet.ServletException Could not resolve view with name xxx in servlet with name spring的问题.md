---
title: 解决Spring MVC报错 javax.servlet.ServletException Could not resolve view with name xxx in servlet with name spring的问题
tags: [Spring MVC, 问题解决]
---

## 问题描述

```
javax.servlet.ServletException: Could not resolve view with name 'xxx' in servlet with name 'spring'
```

## 问题原因

SpringMVC认为是需要请求视图页面

## 解决方法

将对应的请求方法加上`@ResponseBody`
可以注明并不是请求页面