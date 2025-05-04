---
title: 解决Spring注入报错org.springframework.beans.factory.BeanNotOfRequiredTypeException的问题
tags: [Spring, 问题解决]
---

## 问题描述

注入如下bean
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240404085005.png)


报错
*org.springframework.beans.factory.BeanNotOfRequiredTypeException*  : Bean named 'defaultStartProcessOper' must be of type [com.todaytech.pwp.workflow.facade.DefaultStartProcessOper], but was actually of type [$Proxy61]

注入的bean如下
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240404085112.png)

## 解决方法

将注入的属性的类，改为接口，不是具体的类

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240404085118.png)


Spring动态代理的实现方式决定如果要注入的类是实现了接口的，注入的时候只能注入到接口，不可注入到具体的类