---
title: Spring Cloud Gateway使用
tags: [软件开发, Java, Spring, Spring Cloud, Spring Cloud Gateway, 分布式系统]
---

## 项目依赖

项目是Spring Cloud项目，且使用Nacos作为注册中心。Spring Cloud版本为2021.0.5，Spring Cloud Alibaba版本为2021.0.5.0

> [Spring Cloud项目起步配置](https://blog.oliverclio.com/2018/05/27/Spring-Cloud%E9%A1%B9%E7%9B%AE%E8%B5%B7%E6%AD%A5%E9%85%8D%E7%BD%AE.html)
> [Nacos使用](https://blog.oliverclio.com/2020/11/15/Nacos%E4%BD%BF%E7%94%A8.html)

```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-bootstrap</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-gateway</artifactId>  
</dependency>
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
</dependency>
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>
```

## Spring Cloud配置

`bootstrap.yml`

```yml
spring:  
  application:  
    name: doubucket-gateway  
  profiles:  
    active: dev
```

`application-dev.yml`

```yml
server:  
  port: 10050  
spring:  
  cloud:  
    nacos:  
      discovery:  
        server-addr: 192.168.88.121:8848  
        username: nacos  
        password: nacosnacosroot
```

## Spring Cloud Gateway配置

```yml
spring:  
  cloud:  
    gateway:  
      routes:  
        - id: doubucket-items  
          uri: lb://doubucket-items  
          predicates:  
            - Path=/doubucket/items/**  
          filters:  
            - StripPrefix=2
        - id: doubucket-users  
          uri: lb://doubucket-users  
          predicates:  
            - Path=/doubucket/users/**  
          filters:  
            - RewritePath=/doubucket/users(?<segment>/?.*), $\{segment}
```

更多路由规则
[https://docs.spring.io/spring-cloud-gateway/docs/3.1.6/reference/html/](https://docs.spring.io/spring-cloud-gateway/docs/3.1.6/reference/html/)
{:.info}