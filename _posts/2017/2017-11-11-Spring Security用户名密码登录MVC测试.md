---
title: Spring Security用户名密码登录MVC测试
tags: [Spring Security, Spring MVC]
---

## 添加依赖

`pom.xml`

```xml
<dependency>  
    <groupId>org.springframework.security</groupId>  
    <artifactId>spring-security-test</artifactId>  
    <scope>test</scope>  
</dependency>
```

## 测试代码

```java
@SpringBootTest  
@ActiveProfiles("test")  
public class FormLoginMvcTest {  
    private MockMvc mockMvc;  
  
    @BeforeEach  
    void setup(WebApplicationContext webApplicationContext) {  
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).apply(SecurityMockMvcConfigurers.springSecurity()).build();  
    }  
  
    @Test  
    void testFormLogin() throws Exception {  
        MockHttpServletRequestBuilder mockHttpServletRequestBuilder = MockMvcRequestBuilders.post("/formLogin").content("username=Still1&password=123").contentType(MediaType.APPLICATION_FORM_URLENCODED);  
        ResultActions resultActions = mockMvc.perform(mockHttpServletRequestBuilder);  
        resultActions.andExpect(MockMvcResultMatchers.status().isOk());  
        ResponseEntity responseEntity = new ResponseEntity();  
        responseEntity.setData("Oliver Chung");  
        String json = JSONObject.toJSONString(responseEntity);  
        resultActions.andExpect(MockMvcResultMatchers.content().json(json));  
    }  
}
```