---
title: Spring声明式缓存动态cacheNames
tags: [Spring Data, Spring Integration]
---

## 代码实例

动态指定指定缓存name

```java
package com.oliverclio.dms.config.redis;  
  
import com.oliverclio.dms.manager.DmsCommonManager;  
import lombok.AllArgsConstructor;  
import org.springframework.cache.Cache;  
import org.springframework.cache.CacheManager;  
import org.springframework.cache.interceptor.CacheOperationInvocationContext;  
import org.springframework.cache.interceptor.CacheResolver;  
import org.springframework.lang.NonNull;  
  
import java.util.Collection;  
import java.util.HashSet;  
import java.util.Set;  
  
@AllArgsConstructor  
public class DynamicCacheNamesCacheResolver implements CacheResolver {  
  
    private CacheManager cacheManager;  
  
    @Override  
    @SuppressWarnings("rawtypes")  
    @NonNull  
    // 使用具体manager对象的entityName作为缓存的name
    public Collection<? extends Cache> resolveCaches(CacheOperationInvocationContext<?> context) {  
        DmsCommonManager target = (DmsCommonManager) context.getTarget();  
        String entityName = target.getEntityName();  
  
        Cache cache = cacheManager.getCache(entityName);  
        Set<Cache> cacheSet = new HashSet<>();  
        cacheSet.add(cache);  
        return cacheSet;  
    }  
}
```

```java
@Bean  
public CacheResolver dynamicCacheNamesCacheResolver(CacheManager cacheManager) {  
    return new DynamicCacheNamesCacheResolver(cacheManager);  
}
```

```java
@Cacheable(cacheResolver = "dynamicCacheNamesCacheResolver")  
public T getById(Integer id) {  
    return baseMapper.selectById(id);  
}
```