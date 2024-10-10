---
title: Spring Security工作原理
tags: [概念原理, Spring Security]
---

## 基本架构

从servlet的角度去分析，Spring Security实质上是一个filter，具体是通过`DelegatingFilterProxy`这个类实现的。`DelegatingFilterProxy`中默认会将Spring Security需要做的工作代理给`FilterChainProxy`这个bean，而`FilterChainProxy`又会将工作代理给第一个符合条件的`SecurityFilterChain`类型的bean

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230604224946.png" width="700px" />

一个`SecurityFilterChain`的bean中可以绑定多个filter，可以绑定自定义的filter，也可以通过配置绑定Spring Security预定义的filter。

例如，当没有对Spring Security作任何的配置时，默认的一个`SecurityFilterChain`按顺序绑定了以下filter

* `WebAsyncManagerIntegrationFilter`
* `SecurityContextPersistenceFilter`
* `HeaderWriterFilter`
* `CsrfFilter`
* `LogoutFilter`
* `UsernamePasswordAuthenticationFilter`
* `DefaultLoginPageGeneratingFilter`
* `DefaultLogoutPageGeneratingFilter`
* `BasicAuthenticationFilter`
* `RequestCacheAwareFilter`
* `SecurityContextHolderAwareRequestFilter`
* `AnonymousAuthenticationFilter`
* `SessionManagementFilter`
* `ExceptionTranslationFilter`
* `FilterSecurityInterceptor`

当配置对所有请求放行时，`SecurityFilterChain`按顺序绑定了以下filter

```java
@Bean  
public SecurityFilterChain formFilterChain(HttpSecurity http) throws Exception {  
    http.authorizeRequests(requests -> requests.anyRequest().permitAll());
    return http.build();  
}
```

* `WebAsyncManagerIntegrationFilter`
* `SecurityContextPersistenceFilter`
* `HeaderWriterFilter`
* `CsrfFilter`
* `LogoutFilter`
* `RequestCacheAwareFilter`
* `SecurityContextHolderAwareRequestFilter`
* `AnonymousAuthenticationFilter`
* `SessionManagementFilter`
* `ExceptionTranslationFilter`
* `FilterSecurityInterceptor`

可以通过注册多个`SecurityFilterChain`的bean，并指定不同的匹配条件（如URL的匹配规则）和优先匹配顺序，以达到在不同的情况下使用不同的安全规则，详情可参考[[Spring Security同时支持用户名密码登录和OAuth登录#认证配置]]

## 认证架构

### 主要的接口或类简介

`SecurityContextHolder` 身份信息入口类，可以通过这个类获取当前的身份信息
`SecurityContext` 包含身份信息，默认是以`ThreadLocal`的形式存储
`Authentication` 表示身份信息的类，其中包括用户信息`Principal`，密码`Credentials`和权限信息`Authorites`

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230604232943.png" width="500px" />

`GrantedAuthority` 表示权限，例如角色
`AuthenticationManager` 负责定义进行身份认证的方式，被与身份认证相关的filter调用
`ProviderManager` 是`AuthenticationManager`的一般实现，通过调用`AuthenticationProvider`进行具体的身份认证
`AuthenticationProvider` 负责进行特定形式的身份认证
`AuthenticationEntryPoint` 向用户索要身份的入口

### Authentication接口用途

表示身份信息的类，主要有以下两种用途

* 存放即将用于身份认证的信息，例如用户名和密码，此时的`authentication`状态为未认证
* 存放已被认证的当前用户身份信息

### ProviderManager身份认证的过程

通过配置可向特定的`ProviderManager`注册多个`AuthenticationProvider`，不同的`AuthenticationProvider`负责特定形式的身份认证，如根据用户名密码认证、根据JWT进行认证等。`AuthenticationProvider`会根据提供的`Authentication`类型判断能否处理当前形式的身份认证。`ProviderManager`根据注册的顺序依次调用各个`AuthenticationProvider`，直到身份认证成功。假如调用完所有的`AuthenticationProvider`都无法认证成功，则会将身份认证工作委托给父`AuthenticationManager`（通常也是一个`ProviderManager`）

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230604235213.png" width="700px" />

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230604235251.png" width="300px" />

### 全局与局部AuthenticationManager

整个Spring Security存在一个全局的`AuthenticationManager`，每个`SecurityFilterChain`有一个局部的`AuthenticationManager`，局部的`AuthenticationManager`的父`AuthenticationManager`是全局的`AuthenticationManager`

全局的`AuthenticationManager`默认注册了一个`DaoAuthenticationProvider`，可处理基于数据库的用户名密码身份认证

局部的`AuthenticationManager`可以在注册`SecurityFilterChain`时配置

## 参考文档

* [https://docs.spring.io/spring-security/reference/servlet/architecture.html](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
* [https://docs.spring.io/spring-security/reference/5.6/servlet/authentication/architecture.html](https://docs.spring.io/spring-security/reference/5.6/servlet/authentication/architecture.html)