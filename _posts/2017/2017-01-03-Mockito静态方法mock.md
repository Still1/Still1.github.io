---
title: Mockito静态方法mock
tags: [软件测试, Java, Mockito, 单元测试]
---

## 依赖配置

取消Spring Boot默认的`mockito-core`依赖，转变为依赖`mockito-inline`

`pom.xml`

```xml
<dependency>  
   <groupId>org.springframework.boot</groupId>  
   <artifactId>spring-boot-starter-test</artifactId>  
   <exclusions>      
       <exclusion>         
           <groupId>org.mockito</groupId>  
           <artifactId>mockito-core</artifactId>  
        </exclusion>
    </exclusions>
    <scope>test</scope>  
</dependency>  
  
<dependency>  
   <groupId>org.mockito</groupId>  
   <artifactId>mockito-inline</artifactId>  
   <scope>test</scope>  
</dependency>
```

## 无参数的静态方法mock

```java
try (MockedStatic<DateUtil> mockedStatic = Mockito.mockStatic(DateUtil.class)) {  
    mockedStatic.when(DateUtil::getNowLocalDateTime).thenReturn(LocalDateTime.of(2022, 12, 11, 11, 30));  
    MockHttpServletRequestBuilder mockHttpServletRequestBuilder = MockMvcRequestBuilders.post("/formLogin").content("username=Still1&password=123").contentType(MediaType.APPLICATION_FORM_URLENCODED);  
    ResultActions resultActions = mockMvc.perform(mockHttpServletRequestBuilder);
}  
```

## 有参数的静态方法mock

```java
MockedStatic<DoubanItemPhotoOssSavePathGenerator> doubanItemPhotoOssSavePathGeneratorMockedStatic = Mockito.mockStatic(DoubanItemPhotoOssSavePathGenerator.class);  
String stubSavePath = "celebrity/1014853.jpg";  
doubanItemPhotoOssSavePathGeneratorMockedStatic.when(() -> DoubanItemPhotoOssSavePathGenerator.generateSavePath(stubPhotoUrl, CELEBRITY_ID, DoubanItemType.CELEBRITY)).thenReturn(stubSavePath);  
  
TaskInput taskInput = new TaskInput(CELEBRITY_URL);  
Celebrity actualCelebrity = celebrityDoubanDataCrawlerService.crawlItem(taskInput);  
  
doubanItemPhotoOssSavePathGeneratorMockedStatic.close();
```