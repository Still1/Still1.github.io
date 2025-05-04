---
title: 解决Spring Boot项目启动报错 Error creating bean with name 'configurationPropertiesBeans'的问题
tags: [Spring Boot, Spring Cloud, 问题解决]
---

## 问题描述

引入spring-cloud-context依赖后 报错Error creating bean with name 'configurationPropertiesBeans'

## 问题原因

Spring Boot 与 Spring Cloud  版本不对应

## 解决方法

查询相应的版本对应信息，依赖正确的版本
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175210.png)
