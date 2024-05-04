---
title: ThreadLocal使用
tags: [Java]
---

## 带初始值

```java
public class SimpleDateFormatUtil {
    private static final ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal  
        = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyyMMdd HH:mm:ss"));

    public static SimpleDateFormat getSimpleDateFormat() {  
        return simpleDateFormatThreadLocal.get();  
    }
}
```

## 不带初始值

```java
public class SessionUsernameHolder {
    private static final ThreadLocal<String> usernameThreadLocal = new ThreadLocal<>();

    public static String getUsername() {  
        return usernameThreadLocal.get();  
    }  
      
    public static void setUsername(String username) {  
        usernameThreadLocal.set(username);  
    }
}
```