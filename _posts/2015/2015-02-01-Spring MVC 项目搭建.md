---
title: Spring MVC 项目搭建
tags: [Java, Spring, Spring MVC, 软件开发]
---

## 环境配置

### maven依赖

```xml
<dependency>  
  <groupId>org.springframework</groupId>  
  <artifactId>spring-webmvc</artifactId>  
  <version>5.2.1.RELEASE</version>  
</dependency>

<dependency>  
  <groupId>javax.servlet</groupId>  
  <artifactId>javax.servlet-api</artifactId>  
  <version>4.0.1</version>  
  <scope>provided</scope>  
</dependency>

<!-- AspectJ表达式支持，可选 -->
<dependency>  
  <groupId>org.aspectj</groupId>  
  <artifactId>aspectjweaver</artifactId>  
  <version>1.9.4</version>  
</dependency>
```

### web.xml

使用Java配置方式时，可以不配置web.xml。但如果从兼容性考虑，某些中间件可能必须配置web.xml，则可以保留一个基础的web.xml

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"  
         version="3.1">  
</web-app>
```

## Java配置

使用Java代码配置代替传统的web.xml配置

```java
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {  
  
    // 指定root上下文配置类
    @Override  
    protected Class<?>[] getRootConfigClasses() {  
        return new Class<?>[] {RootConfig.class};  
    }  

    // 指定servlet上下文配置类
    @Override  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class<?>[] {DispatcherServletConfig.class};  
    }  

    // 指定DispatcherServlet的servlet mapping
    @Override  
    protected String[] getServletMappings() {  
        return new String[] {"/"};  
    }  

    // 配置web.xml的init-param，multipart-config等
    @Override  
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {  
    }  

    // 配置应用于DispatcherServlet的filter
    @Override  
    protected Filter[] getServletFilters() {  
        return new Filter[] {  
        };  
    }  
}
```

继承`AbstractAnnotationConfigDispatcherServletInitializer`是最方便的配置做法，默认配置两个`WebApplicationContext`，这两个上下文的配置Java类通过`getRootConfigClasses`和`getServletConfigClasses`来指定。获取bean时优先从servlet上下文获取，假如找不到bean，则再去root上下文中获取。一般web项目可以只使用servlet上下文

> [Java Web项目架构#MVC](https://blog.oliverclio.com/2015/03/10/Java-Web%E9%A1%B9%E7%9B%AE%E6%9E%B6%E6%9E%84.html#mvc)

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230225113647.png" width="600px" />

Java代码方式注册bean，代替传统的beans.xml配置

```java
@Configuration  
@ComponentScan(basePackages = "com.oc",  
    includeFilters = {  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..service..*"),  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..component..*"),  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..repository..*")  
    })  
public class RootConfig {
    // 在root上下文中显式注册bean
}
```

```java
@Configuration  
@EnableWebMvc  
@ComponentScan(basePackages = "com.oc",  
    includeFilters = {  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..controller..*"),   
    })
public class DispatcherServletConfig {
    // 在servlet上下文中显式注册bean
}
```

## 模板引擎

引入Thymeleaf

```xml
<dependency>  
  <groupId>org.thymeleaf</groupId>  
  <artifactId>thymeleaf-spring5</artifactId>  
  <version>3.0.11.RELEASE</version>  
</dependency>
```

`DispatcherServletConfig.java`注册Thymeleaf相关的三个bean
* `templateResolver`为`templateEngine`服务，接收到`templateEngine`给的视图名称，根据视图模板名，找到相应的视图模板文件
* `templateEngine`为`viewResolver`服务，接收到`viewResolver`给的视图名称，调用`templateResolver`，拿到视图模板文件，解析视图模板文件，渲染视图，渲染数据模型，最终返回渲染后的视图
* `viewResolver`为`controller`服务，接收到`controller`给的视图名称，调用`templateEngine`，拿到处理后的视图，返回给`controller`

```java
@Configuration  
@EnableWebMvc  
@ComponentScan(basePackages = "com.oc",  
    includeFilters = {  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..controller..*"),  
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "com.oc..view..*")  
    })  
public class DispatcherServletConfig implements WebMvcConfigurer, ApplicationContextAware {  
    private ApplicationContext applicationContext;

    @Bean  
    public ITemplateResolver templateResolver() {  
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();  
        templateResolver.setApplicationContext(this.applicationContext);  
        templateResolver.setPrefix("classpath:/template/");  
        templateResolver.setSuffix(".html");  
        templateResolver.setTemplateMode(TemplateMode.HTML);  
        templateResolver.setCacheable(false);  
        templateResolver.setCharacterEncoding("UTF-8");  
        return templateResolver;  
    }  
      
    @Bean  
    public ISpringTemplateEngine templateEngine() {  
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();  
        templateEngine.setTemplateResolver(templateResolver());  
        templateEngine.setEnableSpringELCompiler(true);  
        templateEngine.addDialect(new SpringSecurityDialect());  
        return templateEngine;  
    }  
      
    @Bean  
    public ViewResolver viewResolver() {  
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();  
        viewResolver.setTemplateEngine(templateEngine());  
        viewResolver.setCharacterEncoding("UTF-8");  
        return viewResolver;  
    }  
      
    @Override  
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {  
        this.applicationContext = applicationContext;  
    }
}
```

## 编写controller

```java
@Controller  
public class SignController {
    @RequestMapping(value = "/signIn", method = RequestMethod.GET)  
    public String signIn() {  
        return "signIn";  
    }
}
```

访问形如http://localhost:8080/signIn的URL，则将会返回名称为signIn的视图，该视图对应的文件为classpath:/template/signIn.html，源代码的位置一般为src/main/resource/template/signIn.html

## 静态资源处理

`DispatcherServletConfig`实现`WebMvcConfigurer`，并重写`addResourceHandlers`方法

```java
@Override  
public void addResourceHandlers(ResourceHandlerRegistry registry) {  
    registry.addResourceHandler("/static/common/**").addResourceLocations("classpath:/common/");  
    registry.addResourceHandler("/static/template/**").addResourceLocations("classpath:/template/");  
    registry.addResourceHandler("/static/pic/**").addResourceLocations("classpath:/pic/");  
  
    String userHomePath = System.getProperty("user.home").replaceAll("\\\\", "/");  
    String picturesPath = "file:" + userHomePath + this.picturesPath;  
    registry.addResourceHandler("/static/picture/**").addResourceLocations(picturesPath);  
}
```

## 配置消息转换器

消息转换器作用于即将返回给客户端的响应信息，对响应信息进行处理，例如处理字符编码，处理JSON格式数据等

`DispatcherServletConfig`实现`WebMvcConfigurer`，并重写`configureMessageConverters`方法

```java
@Override  
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {  
    Charset utf8Charset = StandardCharsets.UTF_8;  
    StringHttpMessageConverter converter = new StringHttpMessageConverter(utf8Charset);  
    converters.add(converter);  
  
    MappingJackson2HttpMessageConverter jsonConverter = new MappingJackson2HttpMessageConverter();  
    converters.add(jsonConverter);  
}
```

## 参考文档

* [https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/spring-framework-reference/web.html#mvc-servlet](https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/spring-framework-reference/web.html#mvc-servlet)
* [https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)