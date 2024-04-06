---
title: 问题解决（Spring Cloud）
tags: [软件开发, Java, Spring Cloud] 
---

## bootstrap.yml配置不生效

Spring Boot默认不支持bootstrap配置文件，只支持application配置文件，需要引入spring-cloud-context后才可以生效

IDEA在配置文件未生效时的图标
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175412.png)

IDEA在配置生效后的图标
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175427.png)