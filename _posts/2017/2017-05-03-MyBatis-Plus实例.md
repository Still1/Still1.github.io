---
title: MyBatis-Plus实例
tags: [MyBatis-Plus]
---

## Maven依赖配置

`pom.xml`

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.2</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.13</version>
</dependency>


<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.30</version>
    <scope>runtime</scope>
</dependency>
```

## Spring Boot配置

`application-dev.yml`

```yaml
spring:
  datasource:
    username: root
    password: mysqlrootroot
    url: jdbc:mysql://192.168.88.13:3306/language_trainer?characterEncoding=utf-8&useSSL=true&serverTimeZone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    druid:
      initial-size: 2
      max-active: 5
      min-idle: 2
mybatis-plus:
  global-config:
    db-config:
      table-prefix: t_
  # 可自定义mapper配置的位置，下面为默认值
  mapper-locations: classpath*:/mapper/**/*.xml
```

Spring Boot启动类

```java
package com.oliverclio.languagetrainer;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.oliverclio.languagetrainer.mapper")
public class LanguageTrainerApplication {

	public static void main(String[] args) {
		SpringApplication.run(LanguageTrainerApplication.class, args);
	}

}

```

## 实体类

```java
package com.oliverclio.languagetrainer.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;

@Data
public class Corpus {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String name;
    private Integer parentId;
}

```

## Mapper

```java
package com.oliverclio.languagetrainer.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.oliverclio.languagetrainer.entity.Corpus;
import org.springframework.stereotype.Repository;

@Repository
public interface CorpusMapper extends BaseMapper<Corpus> {
}

```

## QueryWrapper查询

```java
QueryWrapper<Corpus> queryWrapper = Wrappers.query();  
queryWrapper.eq("parent_id", parentId).orderByAsc("sort");  
corpusMapper.selectList(queryWrapper);
```

## LambdaQueryWrapper查询

```java
LambdaQueryWrapper<UserCorpusStat> lambdaQueryWrapper = Wrappers.lambdaQuery();  
lambdaQueryWrapper.eq(UserCorpusStat::getUserId, userId);  
lambdaQueryWrapper.eq(UserCorpusStat::getCorpusId, corpusId);  
userCorpusStatMapper.selectOne(lambdaQueryWrapper);
```

## 分页查询

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));
    return interceptor;
}

@Bean
public ConfigurationCustomizer configurationCustomizer() {
    return configuration -> configuration.setUseDeprecatedExecutor(false);
}
```

```java
LambdaQueryWrapper<Movie> lambdaQueryWrapper = Wrappers.lambdaQuery();
Page<Movie> page = new Page<>();
page.setSize(100);  
// 当前查询页数
page.setCurrent(1);  
Page<Movie> moviePage = movieMapper.selectPage(page, lambdaQueryWrapper);  
List<Movie> records = moviePage.getRecords();
```

## Mapper配置文件位置

mapper配置文件默认可放置在对应Mapper类的同一目录下，或放置在`classpath*:/mapper/**/*.xml`下。mapper配置文件位置可通过`mapper-locations`属性指定，默认为`classpath*:/mapper/**/*.xml`。

