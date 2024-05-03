---
title: Spring Boot自定义配置属性
tags: [Spring Boot]
---

## 代码实例

### Maven依赖

```xml
<dependency>  
   <groupId>org.springframework.boot</groupId>  
   <artifactId>spring-boot-configuration-processor</artifactId>  
   <optional>true</optional>  
</dependency>
```

### yaml定义自定义属性

```yml
language-trainer:  
  login:  
    oauth-enabled: false
```

### 定义关联属性类

```java
@ConfigurationProperties(prefix = "language-trainer.login")  
@Data  
public class LoginProperties {  
    private boolean oauthEnabled;  
}
```

### Spring Boot启动类配置开启自定义属性

```java
@SpringBootApplication  
@ConfigurationPropertiesScan("com.oliverclio.languagetrainer.config.properties")  
public class LanguageTrainerApplication {  
   public static void main(String[] args) {  
      SpringApplication.run(LanguageTrainerApplication.class, args);  
   }  
}
```

### 注入bean

加上`@Component`注解，在需要使用的地方以`@Autowired`的方式注入即可

```java
@ConfigurationProperties(prefix = "language-trainer.login")
@Component
@Data  
public class LoginProperties {  
    private boolean oauthEnabled;  
}
```

```java
@Autowired
private LoginProperties loginProperties;
```

### 注入静态变量

使用`ApplicationRunner`，在容器初始化完成后执行初始化工作，把自定义配置映射的bean注入到静态变量中，之后就可以通过静态变量的方式使用

```java
@Component  
@AllArgsConstructor  
public class StaticPropertiesInit implements ApplicationRunner {  
  
    private final LoginProperties loginProperties;  
  
    @Override  
    public void run(ApplicationArguments args) {  
        StaticProperties.loginProperties = loginProperties;  
    }  
}
```

注入静态变量的配置假如要用于静态初始化，需要相当小心，必须注意静态初始化的时机，静态初始化必须在容器初始化完成后
{:.warning}

## 消除IDEA找不到自定义属性的警告

通过Maven重新编译即可

```shell
mvn clean compile
```

查看项目编译内容，会生成一个JSON文件`target/classes/META-INF/spring-configuration-metadata.json`，IDEA根据该文件关联配置属性与映射的对象属性

## 参考文档

* [https://blog.csdn.net/qq_33454058/article/details/108482024](https://blog.csdn.net/qq_33454058/article/details/108482024)
* [https://juejin.cn/post/7119368161475952670](https://juejin.cn/post/7119368161475952670)