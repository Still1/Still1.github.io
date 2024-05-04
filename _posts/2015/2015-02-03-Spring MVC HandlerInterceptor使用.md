---
title: Spring MVC HandlerInterceptor使用
tags: [Spring MVC]
---

## 代码实例

`HandlerInterceptor`对`Controller`类的处理方法进行拦截，`preHandle`方法在处理方法调用前执行，`postHandle`方法在处理方法调用后，渲染视图前执行，`afterCompletion`方法在渲染视图完成后执行

```java
@Component  
@AllArgsConstructor  
public class BusinessInterfacePermissionHandlerInterceptor implements HandlerInterceptor {  
  
    private final RoleBusinessInterfaceService roleBusinessInterfaceService;  
  
    @Override  
    @SuppressWarnings("NullableProblems")  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {  
        if (!(handler instanceof HandlerMethod)) {  
            // 继续调用Controller的处理方法
            return true;  
        }  
        boolean hasDmsInterfacePermission = ((HandlerMethod) handler).hasMethodAnnotation(DmsInterfacePermission.class);  
        if (!hasDmsInterfacePermission) {  
            return true;  
        }  
  
        String methodName = handler.toString();  
        Set<Integer> currentUserRoleIdSet = DmsSecurityUtils.getCurrentUserRoleIdSet();  
        for (Integer roleId : currentUserRoleIdSet) {  
            Set<String> businessInterfaceMethodNameSet = roleBusinessInterfaceService.getBusinessInterfaceMethodNameSetByRoleId(roleId);  
            for (String businessInterfaceMethodName : businessInterfaceMethodNameSet) {  
                if (methodName.equals(businessInterfaceMethodName)) {  
                    return true;  
                }  
            }  
        }  
        ResponseUtils.write(response, ResultCode.FORBIDDEN.getCode(),ResultCode.FORBIDDEN.getMsg(),null);  
        // 不继续调用Controller的处理方法
        return false;    
    }  
}
```

```java
@Configuration  
@AllArgsConstructor  
public class WebMvcConfig implements WebMvcConfigurer {  
    private final BusinessInterfacePermissionHandlerInterceptor businessInterfacePermissionHandlerInterceptor;  
  
    @Override  
    public void addInterceptors(InterceptorRegistry registry) {  
        registry.addInterceptor(businessInterfacePermissionHandlerInterceptor);  
    }  
}
```