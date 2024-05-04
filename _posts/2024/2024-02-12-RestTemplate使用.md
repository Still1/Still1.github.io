---
title: RestTemplate使用
tags: [Spring]
---

## 底层选择

RestTemplate只做封装，没有自己实现的底层。发送HTTP请求是同步阻塞式

源码`HttpAccessor`

`ClientHttpRequestFactory`默认是`SimpleClientHttpRequestFactory`，底层使用JDK的`HttpURLConnection`，不支持HTTP的PATCH方法。

底层切换为okHTTP

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.7.2</version>
</dependency>
```

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate(new OkHttp3ClientHttpRequestFactory());
}
```

底层切换为HTTP Client

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.12</version>
</dependency>
```

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate(new HttpComponentsClientHttpRequestFactory());
}
```

## 发送请求

```java
@Autowired
private RestTemplate restTemplate;

public void test() {
    String url = "http://jsonplaceholder.typicode.com/posts/1";
    String body = restTemplate.getForObject(url, String.class);
}
```

传递参数
```java
String url = "http://jsonplaceholder.typicode.com/{type}/{id}";
String body = restTemplate.getForObject(url, String.class, "posts", 1);
```

`getForObject`只返回响应体的封装，`getForEntity`返回完整响应封装

```java
String url = "http://jsonplaceholder.typicode.com/posts/1";
ResponseEntity<String> responseEntity = restTemplate.getForEntity(url, String.class);

// 响应体
String body = responseEntity.getBody();

// 状态码
responseEntity.getStatusCode();
responseEntity.getStatusCodeValue();

// 响应头
responseEntity.getHeaders();
```

POST请求使用`postForObject`和`postForEntity`方法

请求体是JSON格式
```java
String url = "http://jsonplaceholder.typicode.com/posts";

PostDto post = new PostDto();
post.setUserId(1);
post.setTitle("title");
post.setBody("body");

String result = restTemplate.postForObject(url, post, String.class);
```

请求体是URL encoded格式，请求数据指定请求头和请求体
```java
String url = "http://jsonplaceholder.typicode.com/posts";
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
body.add("title", "title");
body.add("body", "body");

HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(body, headers);
String result = restTemplate.postForObject(url, request, String.class);
```

PUT请求
```java
String url = "http://jsonplaceholder.typicode.com/posts/1";
PostDto post = new PostDto();
post.setUserId(1);
post.setTitle("title2");
post.setBody("body2");
restTemplate.put(url, post);
```

DELETE请求
```java
String url = "http://jsonplaceholder.typicode.com/posts/1";
restTemplate.delete(url);
```

HEAD请求
```java
String url = "http://jsonplaceholder.typicode.com/posts/1";
HttpHeaders headers = restTemplate.headForHeaders(url);
```

OPTIONS请求
```java
String url = "http://jsonplaceholder.typicode.com/posts/1";
Set<HttpMethod> options = restTemplate.optionsForAllow(url);
```

通用发送请求
```java
String url = "http://jsonplaceholder.typicode.com/posts";
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
body.add("title", "title");
body.add("body", "body");

HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(body, headers);
// 使用postForEntity方法可以达到同样的效果
ResponseEntity<String> responseEntity = restTemplate.exchange(url, HttpMethod.POST, request, String.class);
```

最通用的发送请求
```java
restTemplate.execute(url, HttpMehtod.GET, request -> {
    // 处理request
}, response -> {
    // 处理response，封装数据
});
```

文件上传
```java
String url = "http://localhost:8080/upload";
FileSystemResource fileSystemResource = new FileSystemResource(Paths.get("C:/test.txt"));
MultiValueMap<String, Object> param = new LinkedMultiValueMap<>();
param.add("file", fileSystemResource);
String result = restTemplate.postForObject(url, param, String.class);
```

文件下载
```java
String url = "http://localhost:8080/download/1";
restTemplate.execute(url, HttpMethod.GET, null, response -> {
    InputStream body = response.getBody();
    Files.copy(body, Paths.get("C:/download.txt"));
});
```

## 错误响应处理

默认处理`DefaultResponseErrorHandler`，实现`ResponseErrorHandler`接口作自定义处理

```java
restTemplate.setErrorHandler(new MyResponseErrorHandler());
```

## 超时

超时时间配置
```java
@Bean
public RestTemplate restTemplate() {
    SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
    requestFactory.setConnectTimeout(10 * 1000);  
    requestFactory.setReadTimeout(10 * 1000);
    return new RestTemplate(factory);
}
```

## 重试

`RestTemplate`整合`RetryTemplate`实现重试。默认不重试

## 负载均衡

`RestTemplate`可以使用`@LoadBalanced`整合Ribbon实现负载均衡