---
title: Spring Boot使用fastjson2作为默认解析器
tags: [软件开发, Java, Spring, Spring Boot, fastjson]
---

## Maven依赖配置

```xml
<dependency>  
   <groupId>com.alibaba.fastjson2</groupId>  
   <artifactId>fastjson2</artifactId>  
   <version>2.0.23</version>  
</dependency>  
<dependency>  
   <groupId>com.alibaba.fastjson2</groupId>  
   <artifactId>fastjson2-extension-spring5</artifactId>  
   <version>2.0.23</version>  
</dependency>
```

## 注册HttpMessageConverters

```java
@Bean  
public HttpMessageConverters fastJsonHttpMessageConverters() {  
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();  
    return new HttpMessageConverters(fastJsonHttpMessageConverter);  
}
```

## 参考文档

[https://github.com/alibaba/fastjson2/blob/main/docs/spring_support_cn.md](https://github.com/alibaba/fastjson2/blob/main/docs/spring_support_cn.md)