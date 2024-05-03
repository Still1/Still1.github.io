---
title: Element UI使用
tags: [Element UI]
---

## Element UI安装

```shell
npm i element-ui -S
```

## 引入Vue2项目

`main.js`

```js
import Vue from 'vue'  
  
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
  
Vue.use(ElementUI)
```