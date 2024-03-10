---
title: Spring容器ApplicationContext放到静态变量
tags: [Spring, Java, 软件开发, Spring Boot]
---

## 设置方式

### Spring Boot启动类设置

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

### 容器事件处理程序设置

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

### ApplicationContextAware设置

```java
@Component
public class ApplicationContextHolder implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {  
        ApplicationContextHolder.applicationContext = applicationContext;  
    }
    
    public static <T> T getBean(Class<T> clazz) {  
        return applicationContext.getBean(clazz);  
    }

    public static Object getBean(String beanName) {  
        return applicationContext.getBean(beanName);  
    }
}
```

### volatile问题

尚未确认设置`applicationContext`的操作是否与后续的读取有happens-before关系，保险的话可以为变量加上`volatile`

```java
private static volatile ApplicationContext applicationContext;
```

### bean创建次序问题

当需要保证`ApplicationContextHolder`优先被创建时，需要使用如`@Order(value = Integer.MIN_VALUE)`的方式保证优先级，数字越小优先级越高。

