---
title: Spring Security OAuth第三方登录实例
tags: [软件开发, Java, Spring, Spring Security, OAuth, 网络安全, JavaScript, Vue, axios]
---

本文例子基于前后端分离项目
{:.info}



## 登录功能实现

### 登录时序图

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230221193757.png){:width="800px"}

### 后端关键代码

基于Spring Boot实现

`pom.xml`引入Sprint Boot OAuth Client

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

`application.yml`

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            clientId: c2d09789aad2
            clientSecret: b300def96a469a08a5774dfb5f9d64
            redirect-uri: http://localhost:8081/#/loginProcess
```

`SecurityConfiguration.java`

```java
@Slf4j
@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests((requests) -> requests.antMatchers("/loginProcess", "/redirect").permitAll().anyRequest().authenticated())
            .oauth2Login()
            .successHandler((request, response, authentication) -> {
                response.setContentType("application/json;charset=utf-8");
                response.setStatus(HttpServletResponse.SC_OK);
                PrintWriter printWriter = response.getWriter();
                String s = JSONObject.toJSONString(authentication.getPrincipal());
                printWriter.write(s);
                printWriter.flush();
                printWriter.close();
            })
            .failureHandler(((request, response, exception) -> {
                response.setContentType("application/json;charset=utf-8");
                response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
                PrintWriter printWriter = response.getWriter();
                printWriter.write(exception.getMessage());
                printWriter.flush();
                printWriter.close();
            }))
            .and().exceptionHandling().authenticationEntryPoint((request, response, authException) -> {
                response.setContentType("application/json;charset=utf-8");
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                PrintWriter printWriter = response.getWriter();
                printWriter.write(authException.getMessage());
                printWriter.flush();
                printWriter.close();
            });

        return http.build();
    }
}
```


### 前端关键代码

`LoginProcessPage.vue`

```vue
<template>
  <h3>Wait a second...</h3>
</template>

<script>
import utils from '@/utils/language-trainer'
export default {
  name: "LoginProcessPage",
  methods: {
    handleLogin() {
      const url = window.location.href
      const code = utils.getUrlQueryParameter(url, 'code')
      const state = utils.getUrlQueryParameter(url, 'state')

      this.axios.get('api/login/oauth2/code/github?code=' + code + '&state=' + state, {
        timeout: 5000,
      }).then(response => {
        window.sessionStorage.setItem('nickname', response.data.attributes.name)
        this.$router.push('/')
        this.$message({
          message: 'Login with GitHub successfully.',
          type: 'success'
        })
      }).catch(() => {
        this.$router.push('/login')
        this.$message({
          message: 'Fail to login with GitHub.',
          type: 'error'
        })
      })
    }
  },
  created() {
    this.handleLogin()
  }
}
</script>

<style scoped>

</style>
```

## 记录登录用户的信息

首次使用OAuth登录时，记录用户信息，相当于注册功能

### 关键代码

```java
http.authorizeRequests((requests) -> requests.antMatchers("/loginProcess", "/redirect").permitAll().anyRequest().authenticated())  
    .oauth2Login()  
    .userInfoEndpoint().userService(userRequest -> {  
        DefaultOAuth2UserService defaultOAuth2UserService = new DefaultOAuth2UserService();  
        OAuth2User oAuth2User = defaultOAuth2UserService.loadUser(userRequest);  
        checkRequiredAttribute(oAuth2User, userRequest);  
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();  
        String userNameAttribute = oAuth2User.getAttributes().get(userNameAttributeName).toString();  
        String registrationId = userRequest.getClientRegistration().getRegistrationId();  
        if (checkFirstLogin(userNameAttribute, registrationId)) {  
            String username;  
            String nickname;  
            if (OauthConstant.GITHUB_TYPE.equals(registrationId)) {  
                String login = oAuth2User.getAttribute(OauthConstant.GITHUB_DEFAULT_USERNAME_KEY);  
                String name = oAuth2User.getAttribute(OauthConstant.GITHUB_DEFAULT_NICKNAME_KEY);  
                username = login;  
                nickname = StringUtils.isNotBlank(name) ? name : login;  
            } else {  
                username = userNameAttribute;  
                nickname = userNameAttribute;  
            }  
            int userId = userService.addUser(username, nickname);  
            UserOauth userOauth = new UserOauth();  
            userOauth.setUserId(userId);  
            userOauth.setOauthUsername(userNameAttribute);  
            userOauth.setOauthType(registrationId);  
            userService.addUserOauth(userOauth);  
        }  
        return oAuth2User;  
    })
```

## 文档参考

* [授权 OAuth 应用](https://docs.github.com/cn/developers/apps/building-oauth-apps/authorizing-oauth-apps)
* [Spring Boot and OAuth2](https://spring.io/guides/tutorials/spring-boot-oauth2/#github-application-config)