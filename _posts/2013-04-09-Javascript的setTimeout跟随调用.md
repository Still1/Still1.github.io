---
title: JavaScript的setTimeout跟随调用
tags: 
  - JavaScript
modify_date: 2013-04-09
---

在Javascript中，如需要在一个函数调用结束后调用另外一个函数，可以使用

<!--more-->

```
setTimeout(function() {}, 0);
```

将此函数加入线程队列，确保前一个函数结束后才会调用这个函数

