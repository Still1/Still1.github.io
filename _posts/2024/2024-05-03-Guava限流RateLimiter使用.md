---
title: Guava限流RateLimiter使用
tags: [Guava]
---

## 代码实例

```java
package com.tinktek.interfaces;  
  
import java.lang.annotation.*;  
  
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface RateLimit {  
    String key() default "";  
    double permitsPerSecond() default 1.0;  
    long timeoutMillis() default 100;  
    String failMessage() default "";  
}
```

```java
package com.tinktek.webapi.common.aspect;  
  
import com.google.common.util.concurrent.RateLimiter;  
import com.tinktek.interfaces.RateLimit;  
import com.tinktek.util.http.ResponseUtils;  
import com.tinktek.webapi.common.enums.ResultCode;  
import lombok.SneakyThrows;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Pointcut;  
import org.aspectj.lang.reflect.MethodSignature;  
import org.springframework.stereotype.Component;  
import org.springframework.web.context.request.RequestContextHolder;  
import org.springframework.web.context.request.ServletRequestAttributes;  
  
import javax.servlet.http.HttpServletResponse;  
import java.lang.reflect.Method;  
import java.util.Map;  
import java.util.concurrent.ConcurrentHashMap;  
import java.util.concurrent.TimeUnit;  
  
@Aspect  
@Component  
@SuppressWarnings("UnstableApiUsage")  
public class RateLimitAspect {  
  
    private final Map<String, RateLimiter> rateLimiterMap = new ConcurrentHashMap<>();  
  
    @Pointcut("@annotation(com.tinktek.interfaces.RateLimit)")  
    public void rateLimitPointCut() {}  
  
    @Around(value = "rateLimitPointCut()")  
    @SneakyThrows  
    public Object around(ProceedingJoinPoint proceedingJoinPoint) {  
        MethodSignature methodSignature = (MethodSignature) proceedingJoinPoint.getSignature();  
        Method method = methodSignature.getMethod();  
        RateLimit rateLimitAnnotation = method.getAnnotation(RateLimit.class);  
  
        String key = rateLimitAnnotation.key();  
        double permitsPerSecond = rateLimitAnnotation.permitsPerSecond();  
  
        if (!rateLimiterMap.containsKey(key)) {  
            rateLimiterMap.putIfAbsent(key, RateLimiter.create(permitsPerSecond));  
        }  
        RateLimiter rateLimiter = rateLimiterMap.get(key);  
  
        long timeoutMillis = rateLimitAnnotation.timeoutMillis();  
        boolean acquire = rateLimiter.tryAcquire(timeoutMillis, TimeUnit.MILLISECONDS);  
        if (!acquire) {  
            ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();  
            if (servletRequestAttributes != null) {  
                String failMessage = rateLimitAnnotation.failMessage();  
                HttpServletResponse response = servletRequestAttributes.getResponse();  
                ResponseUtils.write(response, ResultCode.SUCCESS.getCode(), failMessage,null);  
            }  
            return null;  
        } else {  
            return proceedingJoinPoint.proceed();  
        }  
    }  
}
```

在需要限流的方法上加上`@RateLimit`
```java
@RateLimit(key = "liveStart", permitsPerSecond = 0.3, failMessage = "开启直播中，请稍候。")
```