---
title: Spring Caching KeyGenerator配置
tags: [Spring Data]
---

## 代码实例

指定缓存key的生成策略

### 设置默认KeyGenerator

```java
@Configuration  
public class RedisConfig extends CachingConfigurerSupport {
    @Override  
    public KeyGenerator keyGenerator() {  
        return new SimpleKeyGenerator() {  
            @Override  
            @NonNull        
            public Object generate(@NonNull Object target, @NonNull Method method, @NonNull Object... params) {  
                String originalKey = super.generate(target, method, params).toString();  
                Integer currentUserOrganizationId = DmsSecurityUtils.getCurrentUserOrganizationId();  
                return originalKey + "-" + currentUserOrganizationId;  
            }  
        };  
    }
}
```

### 自定义KeyGenerator

```java
@Component  
public class DmsKeyGenerator extends SimpleKeyGenerator {  
    @Override  
    public Object generate(Object target, Method method, Object... params) {  
        String originalKey = super.generate(target, method, params).toString();  
        Integer currentUserOrganizationId = DmsSecurityUtils.getCurrentUserOrganizationId();  
        return originalKey + "-" + currentUserOrganizationId;  
    }  
}
```

```java
@Cacheable(value = "organizationRangeIdSet", keyGenerator = "dmsKeyGenerator")
```