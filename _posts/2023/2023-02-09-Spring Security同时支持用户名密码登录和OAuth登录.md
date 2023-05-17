---
title: Spring Security同时支持用户名密码登录和OAuth登录
tags: [软件开发, Java, Spring, Spring Security, 网络安全, OAuth]
---

## 前置

先基于Spring Boot实现OAuth登录

> [Spring Security OAuth第三方登录实例](https://blog.oliverclio.com/2022/10/06/Spring-Security-OAuth%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95%E5%AE%9E%E4%BE%8B.html)

## Spring Security配置

### 认证配置

配置两个`SecurityFilterChain`的bean，分别负责OAuth登录和FormLogin的认证设置。两个`SecurityFilterChain`的优先级通过`@Order`指定，优先级较高且符合匹配规则的负责处理此次请求。此处`oauthFilterChain`优先级较高，不以`/formLogin`开头的请求符合匹配规则，都由`oauthFilterChain`负责处理。以`/formLogin`开头的请求则落到`formFilterChain`处理。

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230130062122.png)

`SecurityConfiguration.java`

```java
@Bean  
@Order(2)  
public SecurityFilterChain formFilterChain(HttpSecurity http) throws Exception {  
    http.authorizeRequests((requests) -> requests.anyRequest().authenticated()).formLogin().loginProcessingUrl("/formLogin")  
        .successHandler((request, response, authentication) -> {  
            User principal = (User) authentication.getPrincipal();  
            String nickname = principal.getNickname();  
            ResponseEntity responseEntity = new ResponseEntity();  
            responseEntity.setData(nickname);  
            LanguageTrainerUtil.writeJsonToResponse(response, responseEntity, HttpServletResponse.SC_OK);  
        })  
        .failureHandler(((request, response, exception) -> {  
            ResponseEntity responseEntity = new ResponseEntity();  
            responseEntity.setMessage(exception.getMessage());  
            LanguageTrainerUtil.writeJsonToResponse(response, responseEntity, HttpServletResponse.SC_UNAUTHORIZED);  
        })).and().csrf().disable();  
    return http.build();  
}  
  
@Bean  
@Order(1)  
public SecurityFilterChain oauthFilterChain(HttpSecurity http) throws Exception {  
    // 只验证任何不以/formLogin开头的请求
    http.regexMatcher("^(?!/formLogin).*$").authorizeRequests((requests) -> requests.anyRequest().authenticated())  
        // OAuth登录相关配置
        .oauth2Login()  
        // 获取用户信息处理
        .userInfoEndpoint().userService(userRequest -> {  
            DefaultOAuth2UserService defaultOAuth2UserService = new DefaultOAuth2UserService();  
            OAuth2User oAuth2User = defaultOAuth2UserService.loadUser(userRequest);  
            checkOauthRequiredAttribute(oAuth2User, userRequest);  
            String oauthUsername = oAuth2User.getName();  
            String oauthType = userRequest.getClientRegistration().getRegistrationId();  
            Map<String, Object> oauthAttributes = oAuth2User.getAttributes();  
            User user;  
            if (checkOauthFirstLogin(oauthUsername, oauthType)) {  
                user = registerUserFromOauth(oauthUsername, oauthType, oauthAttributes);  
            } else {  
                user = userService.getUserByOauth(oauthUsername, oauthType);  
            }  
            return user;  
        })  
        .and() 
        // 登录成功处理 
        .successHandler((request, response, authentication) -> {  
            User principal = (User) authentication.getPrincipal();  
            String nickname = principal.getNickname();  
            ResponseEntity responseEntity = new ResponseEntity();  
            responseEntity.setData(nickname);  
            LanguageTrainerUtil.writeJsonToResponse(response, responseEntity, HttpServletResponse.SC_OK);  
        })  
        // 登录失败处理
        .failureHandler(((request, response, exception) -> {  
            ResponseEntity responseEntity = new ResponseEntity();  
            responseEntity.setMessage(exception.getMessage());  
            LanguageTrainerUtil.writeJsonToResponse(response, responseEntity, HttpServletResponse.SC_UNAUTHORIZED);  
        }))  
        // 异常处理，包括登录失败
        .and().exceptionHandling().authenticationEntryPoint((request, response, authException) -> {  
            ResponseEntity responseEntity = new ResponseEntity();  
            responseEntity.setMessage(authException.getMessage());  
            LanguageTrainerUtil.writeJsonToResponse(response, responseEntity, HttpServletResponse.SC_UNAUTHORIZED);  
        }).and().csrf().disable();  
    return http.build();  
}
```

### 自定义用户信息服务

用于FormLogin认证模式的用户信息获取

```java
@Bean  
public UserDetailsService userDetailsService() {  
    return username -> {  
        User user = userService.getUserByUsername(username);  
        if (user == null) {  
            throw new UsernameNotFoundException("User:" + username + " not found");  
        }  
        return user;  
    };  
}
```

## 用户信息

### 实体类

`User`类既是实体类，也同时实现`UserDetails`和`OAuth2User`接口，分别用于FormLogin和OAuth两种认证模式，且作为Spring Security存放用户Pricipal信息的类型

`User.java`

```java
@Data  
@NoArgsConstructor  
public class User implements UserDetails, OAuth2User {  
    @TableId(type = IdType.AUTO)  
    private Integer id;  
    private String username;  
    private String password;  
    private String nickname;  
  
    public User(String username, String nickname) {  
        this.username = username;  
        this.nickname = nickname;  
    }  
  
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        return null;  
    }  
  
    @Override  
    public boolean isAccountNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isAccountNonLocked() {  
        return true;  
    }  
  
    @Override  
    public boolean isCredentialsNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isEnabled() {  
        return true;  
    }  
  
    @Override  
    public Map<String, Object> getAttributes() {  
        return null;  
    }  
  
    @Override  
    public String getName() {  
        return this.username;  
    }  
}
```

## 参考文档

* [https://docs.spring.io/spring-security/reference/5.7/servlet/architecture.html#servlet-filterchainproxy](https://docs.spring.io/spring-security/reference/5.7/servlet/architecture.html#servlet-filterchainproxy)