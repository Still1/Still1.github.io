---
title: Spring Cloud项目起步配置
tags: [Spring Cloud]
---

## 版本配置

获取Spring Cloud版本配置

```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-dependencies</artifactId>  
    <version>2021.0.5</version>  
    <type>pom</type>  
    <scope>import</scope>  
</dependency>  
```

如需要，获取Spring Cloud Alibaba版本配置

```xml
<dependencyManagement>
    <dependencies>
        <dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>  
            <version>2021.0.5.0</version>  
            <type>pom</type>  
            <scope>import</scope>  
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 核心起步依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>
```

`spring-cloud-starter`主要依赖`spring-boot-starter`、`spring-cloud-context`和`spring-cloud-commons`

其他更具体的起步依赖如`spring-cloud-starter-bootstrap`、`spring-cloud-starter-gateway`等都会依赖`spring-cloud-starter`

一般项目直接依赖`spring-cloud-starter-bootstrap`即可，在`spring-cloud-starter`基础上增加了bootstrap配置的支持

## 启动类

与Spring Boot项目一致

> [Spring Boot项目起步配置#启动类](https://blog.oliverclio.com/2018/01/18/Spring-Boot%E9%A1%B9%E7%9B%AE%E8%B5%B7%E6%AD%A5%E9%85%8D%E7%BD%AE.html#%E5%90%AF%E5%8A%A8%E7%B1%BB)