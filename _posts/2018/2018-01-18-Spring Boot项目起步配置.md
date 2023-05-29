---
title: Spring Boot项目起步配置
tags: [软件开发, Java, Spring, Spring Boot]
---

## 版本配置

获取某一个版本的Spring Boot的所有起步依赖的版本配置

继承方式

```xml
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>2.6.13</version>  
</parent>
```

依赖方式

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.13</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

## 核心起步依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

`spring-boot-starter`的主要功能为自动配置，其他更具体的起步依赖如`spring-boot-starter-web`等都会依赖`spring-boot-starter`

## 启动类

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```