---
title: 问题解决（Spring Boot）
tags: [Spring Boot, 问题解决]
---

## Spring Boot项目启动报错 Error creating bean with name 'configurationPropertiesBeans'

引入spring-cloud-context依赖后 报错Error creating bean with name 'configurationPropertiesBeans'

原因：Spring Boot 与 Spring Cloud  版本不对应

解决方法：查询相应的版本对应信息，依赖正确的版本
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175210.png)