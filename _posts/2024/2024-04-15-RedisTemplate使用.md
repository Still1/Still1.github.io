---
title: RedisTemplate使用
tags: [Redis, Spring Data]
---

## Maven依赖

`pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 配置

`application.properites`

```properties
spring.redis.host=192.168.1.11
spring.redis.port=6379
spring.redis.database=0
spring.redis.password=xxx
spring.redis.timeout=3000
spring.redis.lettuce.pool.max-active=20
spring.redis.lettuce.pool.max-wait=-1
spring.redis.lettuce.pool.max-idle=5
spring.redis.lettuce.pool.min-idle=0
```

```java
@Configuration
public class LettuceRedisConfig {

	@Bean
	public RedisTemplate<String, Serializable> redisTemplate(LettuceConnectionFactory connectionFactory) {
		RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();
		redisTemplate.setKeySerializer(new StringRedisSerializer());
		redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
		redisTemplate.setConnectionFactory(connectionFactory);
		return redisTemplate;
	}
}
```

## 使用实例

```java
@Autowire
private RedisTemplate<String, String> strRedisTemplate;
@Autowire
private RedisTemplate<String, Serializable> serializableRedisTemplate;

@Test
public void test() {
    strRedisTemplate.opsForValue().set("str", "test");
    String str = strRedisTemplate.opsForValue().get("str");
    User user = new User();
    serializableRedisTemplate.opsForValue().set("user", user);
	User result = serializableRedisTemplate.opsForValue().get("user");
}
```