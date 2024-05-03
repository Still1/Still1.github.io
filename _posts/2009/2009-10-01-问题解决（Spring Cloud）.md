---
title: 问题解决（Spring Cloud）
tags: [Spring Cloud, 问题解决] 
---

## bootstrap.yml配置不生效

Spring Boot默认不支持bootstrap配置文件，只支持application配置文件，需要引入spring-cloud-context后才可以生效

IDEA在配置文件未生效时的图标
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175412.png)

IDEA在配置生效后的图标
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405175427.png)

## Spring Cloud Gateway 路由报错 java.net.UnknownHostException: renren-fast

问题原因：路由配置uri为http://renren-fast
解决方法：lb://renren-fast