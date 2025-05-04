---
title: 解决EasyUI tab使用href属性获取html内容，无法引入js的问题
tags: [EasyUI, 问题解决]
---

## 问题原因

`href`只加载目标URL的html片段

这个特性是由jQuery封装的ajax请求处理机制所决定的，所以目标URL页面里不需要有`html`,`head`,`body`等标签，即使有这些元素，也会被忽略，所以放在`head`标签里面的任何脚本也不会被引入或者执行。

## 解决方法

由于是tab的主页面引入href属性中的页面片段，所以`href`是可以使用主页面的css和js，可以把js放到主页面（未验证）,或把js放到`<body>`的尾部，不放在`<head>