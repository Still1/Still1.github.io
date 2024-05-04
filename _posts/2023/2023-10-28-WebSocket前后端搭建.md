---
title: WebSocket前后端搭建
tags: [WebSocket]
---

## 服务端

### Maven依赖

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-websocket</artifactId>  
</dependency>
```

### WebSocketConfigurer模式

消息处理

```java
package com.oliverclio.websocket.configurer;  
  
import lombok.SneakyThrows;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.web.socket.*;  
  
import java.util.Map;  
import java.util.concurrent.ConcurrentHashMap;  
import java.util.concurrent.atomic.AtomicInteger;  
  
@SuppressWarnings("NullableProblems")  
@Slf4j  
public class TestWebSocketHandler implements WebSocketHandler {  
    public static final Map<String, WebSocketSession> WEB_SOCKET_SESSION_HOLDER = new ConcurrentHashMap<>();  
    public static final AtomicInteger CONNECTING_WEB_SOCKET_SESSION_COUNT = new AtomicInteger();  
  
    @Override  
    public void afterConnectionEstablished(WebSocketSession session) {  
        if (session.getPrincipal() != null) {  
            log.info("websocket connected!");  
            int count = CONNECTING_WEB_SOCKET_SESSION_COUNT.incrementAndGet();  
            String username = session.getPrincipal().getName();  
            WEB_SOCKET_SESSION_HOLDER.put(username, session);  
            log.info("count:" + count + ";size:" + WEB_SOCKET_SESSION_HOLDER.size());  
        }  
    }  
  
    @Override  
    @SneakyThrows    
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) {  
        if (session.getPrincipal() != null) {  
            String payload = message.getPayload().toString();  
            log.info("message:" + payload);  
            String username = session.getPrincipal().getName();  
            TextMessage textMessage = new TextMessage(payload);  
            for (Map.Entry<String, WebSocketSession> entry : WEB_SOCKET_SESSION_HOLDER.entrySet()) {  
                if (!username.equals(entry.getKey())) {  
                    entry.getValue().sendMessage(textMessage);  
                }  
            }  
            log.info("count:" + CONNECTING_WEB_SOCKET_SESSION_COUNT.get() + ";size:" + WEB_SOCKET_SESSION_HOLDER.size());  
        }  
    }  
  
    @Override  
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {  
        log.info("error");  
        log.info(exception.getMessage());  
    }  
  
    @Override  
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {  
        if (session.getPrincipal() != null) {  
            log.info("websocket disconnected!");  
            CONNECTING_WEB_SOCKET_SESSION_COUNT.decrementAndGet();  
            WEB_SOCKET_SESSION_HOLDER.remove(session.getPrincipal().getName());  
            log.info("count:" + CONNECTING_WEB_SOCKET_SESSION_COUNT.get() + ";size:" + WEB_SOCKET_SESSION_HOLDER.size());  
        }  
    }  
  
    @Override  
    public boolean supportsPartialMessages() {  
        return false;  
    }  
}
```

握手拦截器

```java
package com.oliverclio.websocket.configurer;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.http.server.ServerHttpRequest;  
import org.springframework.http.server.ServerHttpResponse;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.server.HandshakeInterceptor;  
  
import java.util.Map;  
  
@Slf4j  
@SuppressWarnings("NullableProblems")  
public class AuthInterceptor implements HandshakeInterceptor {  
    @Override  
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {  
        log.info("before handshake!!");  
        return true;    
    }  

    @Override  
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {  
        log.info("after handshake!!");  
    }  
}
```

注册配置

```java
package com.oliverclio.logger.websocket;  
  
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.socket.config.annotation.EnableWebSocket;  
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;  
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;  
  
@Configuration  
@EnableWebSocket  
public class WebSocketConfig implements WebSocketConfigurer {  
    @Override  
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {  
        registry.addHandler(new TestWebSocketHandler(), "websocket").addInterceptors(new AuthInterceptor());  
    }  
}
```

### WebSocketMessageBrokerConfigurer模式

启动类配置

```java
@SpringBootApplication  
@IntegrationComponentScan("com.oliverclio.websocket")  
public class WebSocketApplication {
    public static void main(String[] args) {  
        SpringApplication.run(WebSocketApplication.class, args);  
    }
}
```

握手处理

```java
package com.oliverclio.websocket.websocketbroker;  
  
import org.springframework.http.server.ServerHttpRequest;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.server.support.DefaultHandshakeHandler;  
  
import java.security.Principal;  
import java.util.Map;  
  
public class DefaultHandshakeHandlerImpl extends DefaultHandshakeHandler {  
    @Override  
    protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {  
        // 确认身份，使用Spring Security鉴权不需重写
        return () -> "xxx";  
    }  
}
```

消息处理，装饰者模式

```java
package com.oliverclio.websocket.websocketbroker;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.web.socket.CloseStatus;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.WebSocketMessage;  
import org.springframework.web.socket.WebSocketSession;  
import org.springframework.web.socket.handler.WebSocketHandlerDecorator;  
  
@Slf4j  
@SuppressWarnings("NullableProblems")  
public class WebSocketBasicHandler extends WebSocketHandlerDecorator {  
    public WebSocketBasicHandler(WebSocketHandler delegate) {  
        super(delegate);  
    }  
  
    @Override  
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {  
        log.info("connected");  
        // 获取身份信息
        log.info(session.getPrincipal().getName());
        getDelegate().afterConnectionEstablished(session);  
    }  
  
    @Override  
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {  
        log.info(message.getPayload().toString());  
        getDelegate().handleMessage(session, message);  
    }  
  
    @Override  
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {  
        log.info("closed");  
        getDelegate().afterConnectionClosed(session, closeStatus);  
    }  
}
```

消息处理，装饰者工厂

```java
package com.oliverclio.websocket.websocketbroker;  
  
import org.springframework.stereotype.Component;  
import org.springframework.web.socket.WebSocketHandler;  
import org.springframework.web.socket.handler.WebSocketHandlerDecoratorFactory;  
  
@Component  
@SuppressWarnings("NullableProblems")  
public class WebSocketHandlerDecoratorFactoryImpl implements WebSocketHandlerDecoratorFactory {  
    @Override  
    public WebSocketHandler decorate(WebSocketHandler handler) {  
        return new WebSocketBasicHandler(handler);  
    }  
}
```

注册配置

```java
package com.oliverclio.websocket.websocketbroker;  
  
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;  
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;  
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;  
import org.springframework.web.socket.config.annotation.WebSocketTransportRegistration;  
  
@Configuration  
@EnableWebSocketMessageBroker  
public class WebSocketMessageBrokerConfig implements WebSocketMessageBrokerConfigurer {  
    @Override  
    public void registerStompEndpoints(StompEndpointRegistry registry) {  
        registry.addEndpoint("/websocket").setHandshakeHandler(new DefaultHandshakeHandlerImpl());  
    }  
  
    @Override  
    public void configureWebSocketTransport(WebSocketTransportRegistration registry) {  
        registry.addDecoratorFactory(new WebSocketHandlerDecoratorFactoryImpl());  
    }  
}
```

### 鉴权

WebSocket通过HTTP进行握手，可对握手请求进行鉴权，依赖对HTTP的鉴权方式，如使用Spring Security

## 客户端

### Vue

```html
<template>  
  <div></div>
</template>  
  
<script>  
export default {  
  name: "WebSocket",  
  mounted() {  
    let ws = new WebSocket("ws://localhost:8080/websocket")  
  
    ws.onopen = function() {  
      console.log("Connection open ...")  
      ws.send("Hello WebSockets!")  
    }  
  
    ws.onmessage = function(evt) {  
      console.log( "Received Message: " + evt.data)  
      ws.close()  
    }  
  
    ws.onclose = function() {  
      console.log("Connection closed.")  
    }  
  }  
}  
</script>  
  
<style scoped>  
</style>
```