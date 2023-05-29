---
title: Spring MVC 集成测试
tags: [软件测试, 集成测试, Java, Spring, Spring MVC]
---

## 独立测试profile

设置测试独立使用的配置，例如测试数据库的连接池配置

`application-test.yml`

```yaml
spring:  
  datasource:  
    username: root  
    password: mysqlrootroot  
    url: jdbc:mysql://192.168.88.13:3306/language_trainer_test?characterEncoding=utf-8&useSSL=true&serverTimeZone=Asia/Shanghai  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    druid:  
      initial-size: 2  
      max-active: 5  
      min-idle: 2
```

## 编写测试代码

```java
package com.oliverclio.languagetrainer.controller;  
  
import com.alibaba.fastjson2.JSON;  
import com.oliverclio.languagetrainer.entity.Corpus;  
import org.junit.jupiter.api.BeforeEach;  
import org.junit.jupiter.api.Test;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.http.MediaType;  
import org.springframework.test.context.ActiveProfiles;  
import org.springframework.test.web.servlet.MockMvc;  
import org.springframework.test.web.servlet.ResultActions;  
import org.springframework.test.web.servlet.request.MockHttpServletRequestBuilder;  
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;  
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;  
import org.springframework.test.web.servlet.setup.MockMvcBuilders;  
import org.springframework.web.context.WebApplicationContext;  
  
import java.util.ArrayList;  
import java.util.List;  
  
@SpringBootTest  
@ActiveProfiles("test")  
public class CorpusControllerMvcTest {  
  
    private MockMvc mockMvc;  
  
    @BeforeEach  
    void setup(WebApplicationContext webApplicationContext) {  
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();  
    }  
  
    @Test  
    void testGetCorpusListByParentId() throws Exception {  
        MockHttpServletRequestBuilder mockHttpServletRequestBuilder = MockMvcRequestBuilders.get("/corpus").param("parentId", "0");  
        ResultActions resultActions = mockMvc.perform(mockHttpServletRequestBuilder);  
        resultActions.andExpect(MockMvcResultMatchers.status().isOk());  
        resultActions.andExpect(MockMvcResultMatchers.content().contentType(MediaType.APPLICATION_JSON));  
  
        List<Corpus> corpusList = new ArrayList<>();  
        Corpus firstCorpus = new Corpus();  
        firstCorpus.setId(1);  
        firstCorpus.setParentId(0);  
        firstCorpus.setName("English");  
        firstCorpus.setSort(1);  
  
        Corpus secondCorpus = new Corpus();  
        secondCorpus.setId(2);  
        secondCorpus.setParentId(0);  
        secondCorpus.setName("French");  
        secondCorpus.setSort(2);  
  
        corpusList.add(firstCorpus);  
        corpusList.add(secondCorpus);  
  
        String json = JSON.toJSONString(corpusList);  
        resultActions.andExpect(MockMvcResultMatchers.content().json(json));  
    }  
}
```

## 验证JSON部分属性

通过JsonPath定位需要验证的属性

```java
resultActions.andExpect(MockMvcResultMatchers.jsonPath("$.data.learningCount").value(2));
```

## 模拟用户登录

使用注解`@WithUserDetails`，通过`UserDetailsService`查询对应用户名的用户信息

> [自定义UserDetailsService](https://blog.oliverclio.com/2023/02/09/Spring-Security%E5%90%8C%E6%97%B6%E6%94%AF%E6%8C%81%E7%94%A8%E6%88%B7%E5%90%8D%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E5%92%8COAuth%E7%99%BB%E5%BD%95.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF%E6%9C%8D%E5%8A%A1)

```java
private static final String DEFAULT_TEST_USER = "Still1";

@Test  
@WithUserDetails(DEFAULT_TEST_USER)  
void testCompleteCorpusLearningMode() throws Exception {  
    testCompleteCorpus(CommonConstant.CorpusMode.LEARNING);  
}
```

## 参考文档

* [https://docs.spring.io/spring-security/reference/5.7/servlet/test/method.html](https://docs.spring.io/spring-security/reference/5.7/servlet/test/method.html)
* [https://docs.spring.io/spring-framework/docs/5.3.25/reference/html/testing.html#spring-mvc-test-server-defining-expectations](https://docs.spring.io/spring-framework/docs/5.3.25/reference/html/testing.html#spring-mvc-test-server-defining-expectations)