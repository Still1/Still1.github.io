---
title: 解决Spring MVC 响应返回中文内容变成问号的问题
tags: [Spring MVC, 问题解决]
---

## 解决方法

`@ResponseBody` 解决方法：
DispatcherServlet XML应用上下文配置
```xml
<bean id="utf8Charset" class="java.nio.charset.Charset" factory-method="forName">
  <constructor-arg value="UTF-8"/>
</bean>

<mvc:annotation-driven>
  <mvc:message-converters>
	<bean class="org.springframework.http.converter.StringHttpMessageConverter">
      <constructor-arg ref="utf8Charset"/>
    </bean>

  </mvc:message-converters>
</mvc:annotation-driven>
```
或Java配置类重写方法
```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    Charset utf8Charset = Charset.forName("UTF-8");
    StringHttpMessageConverter converter = new StringHttpMessageConverter(utf8Charset);
    converters.add(converter);
}
```

控制Content-type

```java
@RequestMapping(value = "/hotel", produces = "application/json; charset=UTF-8")
```

返回HTML网页里包含的中文带问号解决方法

```java
@Bean
public ViewResolver viewResolver(TemplateEngine  templateEngine) {
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine);
    viewResolver.setCharacterEncoding("UTF-8");
    return  viewResolver;
}
```
检查视图解析器的编码设置