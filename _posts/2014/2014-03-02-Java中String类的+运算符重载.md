---
title: Java中String类的+运算符重载
tags: 
  - Java
  - Summary
modify_date: 2014-03-02
---

## String的运算符重载

`String`中的`+`和`+=`是Java中仅有的两个重载运算符

<!--more-->

```java
String s = "a" + "b" + "c";
```

上面代码块相当于

```java
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append("a");
stringBuilder.append("b");
stringBuilder.append("c");
String s = stringBuilder.toString();
```

每次使用`+`或`+=`运算符，都会创建一个`StringBuilder`对象，因此要注意不要在`for`循环内部使用，否则可能会创建过多的`StringBuilder`对象，降低性能