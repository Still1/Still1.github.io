---
title: Spring MVC使用properties配置
tags: [Spring MVC]
---

## 代码实例

在root上下文配置类上加上注解`@PropertySource`

```java
@Configuration  
@PropertySource({  
    "classpath:properties/database.properties",  
    "classpath:properties/path.properties"
})
public class RootConfig {
}
```

`path.properties`

```properties
picturesPath=/greenbean/pictures/
```

`database.properties`

```properties
greenbean.db={\  
  driverClassName : 'com.mysql.cj.jdbc.Driver',\  
  url : 'jdbc:mysql://localhost:3306/greenbean?serverTimezone=GMT%2B8',\  
  username : 'root',\  
  password : 'mysqlroot'\  
}
```

通过注解`@Value`引用配置

```java
// 一般字符串
@Value("${picturesPath}")
private String picturesPath;
```

```java
// JSON字符串解析为Map
// #{}中使用的语法为SpEL
@Value("#{${greenbean.db}}")
private Map<String, String> databaseProperties;
```

`@Value`的作用时机与`@Autowired`类似，只是注入的不是bean而是外部配置。`@Value`可以使用在属性、setter方法、构造方法参数上

## 注入到静态变量

利用IoC容器创建bean的机制，在创建bean时将配置值通过setter方法引入，但setter方法内并非把属性赋值到对象的成员变量中，而是赋值到静态变量中。由于赋值时类加载已完成，静态变量不可再标记为`final`

```java
@Component
public class MyProperties {
    public static String config;

    @Value("${config}")
    public void setConfig(String config) {
        MyProperties.config = config;
    }
}
```


实际上是一种魔改方案，正常情况下不建议使用。建议使用使用另外一种方式，参考[Spring Boot自定义配置属性#注入静态变量](https://blog.oliverclio.com/2018/10/30/Spring-Boot%E8%87%AA%E5%AE%9A%E4%B9%89%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7.html#%E6%B3%A8%E5%85%A5%E9%9D%99%E6%80%81%E5%8F%98%E9%87%8F)
{:.warning}