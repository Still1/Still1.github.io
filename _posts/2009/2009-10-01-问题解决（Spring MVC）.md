---
title: 问题解决（Spring MVC）
tags: [Spring MVC, 问题解决]
---

## 报错 javax.servlet.ServletException: Could not resolve view with name 'xxx' in servlet with name 'spring'

原因：
这是由于SpringMVC认为是需要请求视图页面

解决方法：
将对应的请求方法加上`@ResponseBody`
可以注明并不是请求页面

## Spring MVC 响应返回中文内容变成问号

`@ResponseBody` 解决方法：
DispatcherServlet XML应用上下文配置
```xml
<bean id="utf8Charset" class="java.nio.charset.Charset" factory-method="forName">
  <constructor-arg value="UTF-8"/>
</bean>

<mvc:annotation-driven>
  <mvc:message-converters>
	<bean class="org.springframework.http.converter.StringHttpMessageConverter">
      <constructor-arg ref="utf8Charset"/>
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