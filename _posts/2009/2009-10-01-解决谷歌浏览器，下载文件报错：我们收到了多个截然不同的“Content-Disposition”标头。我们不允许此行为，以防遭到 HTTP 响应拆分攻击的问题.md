---
title: 解决谷歌浏览器，下载文件报错：我们收到了多个截然不同的“Content-Disposition”标头。我们不允许此行为，以防遭到 HTTP 响应拆分攻击的问题
tags: [浏览器, 问题解决]
---

## 问题原因

可能是HTTP头Content-Disposition中的filename带有未转义的引号、逗号等字符

## 解决方法

将filename需要转义的字符转义

IE内核的浏览器不会出现此问题