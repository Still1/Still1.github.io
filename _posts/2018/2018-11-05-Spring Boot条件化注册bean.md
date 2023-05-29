---
title: Spring Boot条件化注册bean
tags: [Java, Spring, Spring Boot, 软件开发]
---

## 项目配置属性的值作为条件

配置的属性满足某个值，才注册这个bean

```yml
language-trainer:  
  login:  
    oauth-enabled: false
```

```java
@Bean  
@ConditionalOnProperty(name = "language-trainer.login.oauth-enabled", havingValue = "true")
```