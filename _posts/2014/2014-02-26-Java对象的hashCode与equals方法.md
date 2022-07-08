---
title: Java对象的hashCode与equals方法
tags: 
  - Java
  - Summary
modify_date: 2014-02-26
---

## hashCode与equals方法的重写原则

<!--more-->

### 唯一原则

`equals`方法判断为`true`，则`hashCode`方法返回的哈希值也必须相等

### 逆否命题

`hashCode`方法返回的哈希值不相等，则`equals`方法必须判断为`false`

### 推论

`hashCode`方法返回的哈希值相等，无法判断`equals`方法的返回结果

`equals`方法判断为`false`，无法判断`hashCode`方法返回的哈希值是否相等