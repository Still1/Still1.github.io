---
title: 解决Maven警告Maven site WARNING No project URL defined - decoration links will not be relativized!的问题
tags: [Maven, 问题解决]
---

## 问题原因

项目POM没有定义URL，Maven建立站点时需要此URL，没有显式定义将根据项目名称生成一个，但项目名称带中划线`-`

## 解决方法

显式在POM定义URL
```xml
<url>http://ocframework.oc.com/core</url>
```