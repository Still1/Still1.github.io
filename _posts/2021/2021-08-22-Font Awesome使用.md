---
title: Font Awesome使用
tags: [Font Awesome]
---

## 安装Font Awesome

```shell
npm i --save @fortawesome/fontawesome-svg-core

# 免费icon，引入需要的icon包
npm i --save @fortawesome/free-solid-svg-icons
npm i --save @fortawesome/free-regular-svg-icons
npm i --save @fortawesome/free-brands-svg-icons

# 安装Vue2组件
npm i --save @fortawesome/vue-fontawesome@latest-2
```

## 引入Vue2项目

`main.js`

```js
import Vue from 'vue'  
  
import { library } from '@fortawesome/fontawesome-svg-core'  
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

// 引入需要的icon
import { faGithub } from '@fortawesome/free-brands-svg-icons'  
import { faUser } from '@fortawesome/free-solid-svg-icons'  
  
library.add(faGithub)  
library.add(faUser)  

Vue.component('font-awesome-icon', FontAwesomeIcon)
```

## 使用icon

```html
<font-awesome-icon icon="fa-brands fa-github" size="2x" />
<font-awesome-icon icon="fa-solid fa-user" />
```