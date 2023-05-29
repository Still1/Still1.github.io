---
title: Spring Security使用
tags: [Java, Spring, Spring Security, 网络安全, 软件开发]
---

本文例子基于前后端不分离项目
{:.info}

## Maven依赖

```xml
<dependency>  
  <groupId>org.springframework.security</groupId>  
  <artifactId>spring-security-web</artifactId>  
  <version>5.2.1.RELEASE</version>  
</dependency>  
  
<dependency>  
  <groupId>org.springframework.security</groupId>  
  <artifactId>spring-security-config</artifactId>  
  <version>5.2.1.RELEASE</version>  
</dependency>
```

## Java配置

```java
@EnableWebSecurity  
public class SecurityConfig extends WebSecurityConfigurerAdapter {  
  
    private static final String QUERY_USER_SQL = "select username, password, enabled from t_user where username = ?";  
    private static final String QUERY_AUTHORITY_SQL = "select username, authority from t_authority inner join t_user on t_authority.user_id = t_user.id where username = ?";  
  
    private DataSource dataSource;  
  
    private AuthenticationSuccessHandler authenticationSuccessHandler;  
  
    @Autowired  
    public SecurityConfig(DataSource dataSource, AuthenticationSuccessHandler authenticationSuccessHandler) {  
        this.dataSource = dataSource;  
        this.authenticationSuccessHandler = authenticationSuccessHandler;  
    }  

    // 表单验证相关配置
    @Override  
    protected void configure(HttpSecurity http) throws Exception {  
        http.authorizeRequests()  
            .antMatchers("/static/common/**", "/static/template/**", "/signUp", "/signUp/*").permitAll()  
            .anyRequest().authenticated()  
            .and().formLogin().loginPage("/signIn").permitAll()  
            .successHandler(authenticationSuccessHandler)  
            .failureUrl("/signInError")  
            .and().logout().logoutUrl("/signOut").logoutSuccessUrl("/signIn")
            .and().rememberMe().key("greenbean").tokenValiditySeconds(259200)
            .authenticationSuccessHandler(authenticationSuccessHandler);
    }  

    // 用户数据来源配置
    @Override  
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {  
        auth.jdbcAuthentication().dataSource(dataSource)  
            .usersByUsernameQuery(QUERY_USER_SQL)  
            .authoritiesByUsernameQuery(QUERY_AUTHORITY_SQL);  
        // 这里可以改为配置userDetailsService
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```

登录成功后处理

```java
@Component  
public class SignInSuccessHandler implements AuthenticationSuccessHandler {  
  
    private UserService userService;  
  
    @Autowired  
    public SignInSuccessHandler(UserService userService) {  
        this.userService = userService;  
    }  
  
    public SignInSuccessHandler() {  
    }  
  
    @Override  
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException {  
        String username = authentication.getName();  
        User user = userService.getUserByUsername(username);  
  
        HttpSession session = httpServletRequest.getSession();  
        session.setAttribute("userId", user.getId());  
        session.setAttribute("userNickname", user.getNickname());  
        session.setAttribute("userAvatar", user.getAvatar());  
  
        httpServletResponse.sendRedirect("home");  
    }  
}
```

## controller配置

```java
@Controller  
public class SignController {  
    @RequestMapping(value = "/signIn", method = RequestMethod.GET)  
    public String signIn() {  
        return "signIn";  
    }  

    // 登录失败则添加提示信息，返回登录页并显示提示信息
    @RequestMapping("/signInError")  
    public String signInError(Model model) {  
        model.addAttribute("signInError", true);  
        return "signIn";  
    }
}
```

## 整合Thymeleaf

### 登录页

```html
<!DOCTYPE html>  
<html lang="en" xmlns:th="http://www.thymeleaf.org">  
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <h1>Login page</h1>
    <p th:if="${loginError}" class="error">Wrong user or password</p>
    <form th:action="@{/login}" action="login" method="post">
        <label for="username">Username</label>:
        <input type="text" id="username" name="username" autofocus="autofocus" /> <br />
        <label for="password">Password</label>:
        <input type="password" id="password" name="password" /> <br />
        <input type="submit" value="Log in" />
        <input type="checkbox" name="remember-me"> Remember me
    </form>
</body>
</html>
```

### 登出

截取主页登出部分代码

```html
<a class="dropdown-item" id="signOutLink" href="javascript:void(0);">Sign out</a>
<!-- 开启CSRF防护后，登出必须是POST请求 -->
<form id="signOutForm" style="display: none" method="post" th:action="@{/signOut}"></form>
```

```js
(function() {  
    $(function() {  
        const signOutLink = $('#signOutLink');  
        signOutLink.on("click", function () {  
            $('#signOutForm').trigger("submit");  
        });  
    });  
})();
```

## 参考文档

* [Spring Security Guides](https://docs.spring.io/spring-security/site/docs/5.1.6.RELEASE/guides/html5)
* [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.1.6.RELEASE/reference/htmlsingle/)
* [Spring Security Documentation](https://spring.io/projects/spring-security#learn)