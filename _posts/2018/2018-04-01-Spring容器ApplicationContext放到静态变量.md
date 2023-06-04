---
title: Spring容器ApplicationContext放到静态变量
tags: [Spring, Java, 软件开发, Spring Boot]
---

## Spring Boot启动类设置

```java
public class ApplicationContextHolder {  
    public static ApplicationContext applicationContext;
}
```

```java
public class DoubucketItemsApplication {  
    public static void main(String[] args) {  
        ApplicationContextHolder.applicationContext = SpringApplication.run(DoubucketItemsApplication.class, args);
    }  
}
```

## 容器事件处理程序设置

```java
@Component  
@AllArgsConstructor  
public class ApplicationContextHolder implements ApplicationRunner {  
    public static ApplicationContext applicationContext;  
  
    private ApplicationContext context;  
  
    @Override  
    public void run(ApplicationArguments args) {  
        applicationContext = context;  
    }  
}
```