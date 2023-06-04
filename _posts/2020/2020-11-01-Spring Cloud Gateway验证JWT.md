---
title: Spring Cloud Gateway验证JWT
tags: [Java, 软件开发, Spring, Spring Cloud, Spring Cloud Gateway]
---

## 无需验证的URI集合配置

`application.yml`
```yml
doubucket:  
  authentication:  
    permitUri:  
      - /users/getToken
```

## 定义全局filter

```java
@Component  
@AllArgsConstructor  
public class AuthenticationFilter implements GlobalFilter, Ordered {  
  
    private final AuthenticationProperties authenticationProperties;  
  
    private final DoubucketJwtUtils doubucketJwtUtils;  
  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
        ServerHttpRequest request = exchange.getRequest();  
        ServerHttpResponse response = exchange.getResponse();  
        String path = request.getPath().toString();  
        Set<String> permitUriList = authenticationProperties.getPermitUri();  
        if (permitUriList.contains(path)) {  
            return chain.filter(exchange);  
        }  
  
        HttpHeaders headers = request.getHeaders();  
        String authorizationHeader = headers.getFirst(HttpHeaders.AUTHORIZATION);  
  
        if (StringUtils.isBlank(authorizationHeader) || !StringUtils.startsWith(authorizationHeader, "Bearer ")) {  
            response.setStatusCode(HttpStatus.UNAUTHORIZED);  
            return response.setComplete();  
        }  
  
        String token = authorizationHeader.split(" ")[1].trim();  
  
        Claims claims = doubucketJwtUtils.parse(token);  
        if (claims != null) {  
            return chain.filter(exchange);  
        } else {  
            response.setStatusCode(HttpStatus.FORBIDDEN);  
            return response.setComplete();  
        }  
    }  
  
    @Override  
    public int getOrder() {  
        return HIGHEST_PRECEDENCE + 1;  
    }  
}
```