---
title: 解决Java String的trim方法无法去掉所有空格的问题
tags: [Java, 问题解决]
---

## 问题描述

使用`String`的`trim()`方法，无法去掉所有空格

## 问题原因

无法去掉的空格可能是中文空格，可以使用`charAt`看看是否是值为12288的字符，如果是，则是中文空格

## 解决方法

```java
char chineseSpace = 12288;
return string.replaceAll(String.valueOf(chineseSpace), "");
```