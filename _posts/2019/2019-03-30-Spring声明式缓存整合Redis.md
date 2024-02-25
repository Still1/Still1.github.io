---
title: Spring声明式缓存整合Redis
tags: [软件开发, Java, Spring, Redis, Spring Data, Spring Integration, 缓存]
---

## 安装Redis

> [Redis在Docker下的安装](https://blog.oliverclio.com/2020/03/05/Redis%E5%9C%A8Docker%E4%B8%8B%E7%9A%84%E5%AE%89%E8%A3%85.html)

## 依赖配置

基于Spring Boot项目

> [Spring Boot项目起步配置](https://blog.oliverclio.com/2018/01/18/Spring-Boot%E9%A1%B9%E7%9B%AE%E8%B5%B7%E6%AD%A5%E9%85%8D%E7%BD%AE.html)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- Lettuce使用连接池时需要添加依赖 -->
<dependency>  
    <groupId>org.apache.commons</groupId> 
    <artifactId>commons-pool2</artifactId>
</dependency>
```

## Redis配置

```yml
spring: 
  redis:
    host: 192.168.88.151
    port: 6379
    password: redisredisroot
    timeout: 10000
    database: 0
    lettuce:
      pool: 
        enabled: true
```

## 注册CacheManager

```java
@Bean  
public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {  
    RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();  
    redisCacheConfiguration.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));  
  
    return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();  
}
```

## 启动类

添加注解`@EnableCaching`

```java
import org.springframework.cache.annotation.EnableCaching;

@EnableCaching
public class MyApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(MyApplication.class, args);  
    }  
}
```

## Cache Aside Pattern实现

利用Spring声明式缓存对缓存的抽象实现，与具体的缓存实现（例如Redis）无关

* 在需要缓存返回值的方法上加上注解`@Cacheable("cacheName")`
* 参与缓存key生成的方法参数，需要正确实现`hashCode`和`equals`方法
* 返回值涉及到的所有类型（包括父类、依赖对象的类型）都需要实现`Serializable`接口
* 在更新缓存相关数据的方法上加上注解`@CacheEvict("cacheName")`

返回值类

```java
@Data
public class Movie implements Serializable {
    private Integer id;
    private String name;
}
```

获取数据方法，使用缓存

```java
@Cacheable("movie")  
public Movie getMovieById(int id) {
    // 从数据库获取数据
}
```

更新数据方法，删除缓存

```java
@CacheEvict(cacheNames = "movie", key = "#movie.id")  
public void updateMovie(Movie movie) {  
    // 更新数据库数据  
}
```

```java
@Caching(evict = {
    @CacheEvict(cacheNames = "organizationRangeIdSet", allEntries = true),
    @CacheEvict(cacheNames = "organization", key = "#entity.id")
})
```

## 缓存注解使用注意

* 缓存注解，如`@Cacheable`,`@CacheEvict`等，实现原理为动态代理，因此具有与声明式事务`@Transactional`类似的失效情况（如类内部调用失效等）
* `@CacheEvict`必须用在返回值为`void`的方法上

## 参考文档

* [https://docs.spring.io/spring-boot/docs/2.6.14/reference/html/data.html#data.nosql.redis](https://docs.spring.io/spring-boot/docs/2.6.14/reference/html/data.html#data.nosql.redis)
* [https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)
* [https://docs.spring.io/spring-data/redis/docs/2.6.10/reference/html/#redis:support:cache-abstraction](https://docs.spring.io/spring-data/redis/docs/2.6.10/reference/html/#redis:support:cache-abstraction)