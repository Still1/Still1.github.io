---
title: Spring Boot项目配置Servlet、Filter、Listener
tags: [软件开发, Java, Spring, Spring Boot]
---

## 代码实例

启动类增加注解`@ServletComponentScan`

```java
@SpringBootApplication  
@ServletComponentScan(value = "com.oliverclio.logger")  
public class EmqxHttpLoggerApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(EmqxHttpLoggerApplication.class, args);  
    }  
}
```

使用`@WebFilter`注解注册一个Filter

```java
@Order(1)
@WebFilter(filterName = "authenticationFilter", urlPatterns = "/*")  
public class AuthenticationFilter implements Filter {  
  
    public static final String SECRET = "ynzin8z@4!.mal";  
    public static final String SECRET_HEADER_NAME = "Emqx-Http-Logger-Secret";  
  
    @Override  
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {  
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;  
        String secret = httpServletRequest.getHeader(SECRET_HEADER_NAME);  
        if (SECRET.equals(secret)) {  
            filterChain.doFilter(servletRequest, servletResponse);  
        }  
    }  
}
```

类似地，可使用`@WebServlet`和`@WebListener`注册Servlet和Listener

需要指定顺序时，可使用`@Order`注解