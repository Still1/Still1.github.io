---
title: Spring Security获取当前登录用户信息
tags: [Spring Security]
---

## 代码实例

通过`ThreadLocal`实现，可以在线程中的任何位置调用

```java
public class SecurityUtil {  
    public static int getCurrentLoginUserId() {  
        User currentLoginUser = getCurrentLoginUser();  
        return currentLoginUser.getId();  
    }  
  
    private static User getCurrentLoginUser() {  
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();  
        return (User) authentication.getPrincipal();  
    }  
}
```