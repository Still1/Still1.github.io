---
title: Spring整合MyBatis
tags: [软件开发, Java, MyBatis, Spring]
---

## maven依赖

```xml
<dependency>  
  <groupId>org.mybatis</groupId>  
  <artifactId>mybatis</artifactId>  
  <version>3.5.2</version>  
</dependency>

<dependency>  
  <groupId>org.mybatis</groupId>  
  <artifactId>mybatis-spring</artifactId>  
  <version>2.0.2</version>  
</dependency>
```

## 注册SqlSessionFactory

```java
@Bean  
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {  
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();  
    sqlSessionFactoryBean.setDataSource(dataSource);  
    org.apache.ibatis.session.Configuration myBatisConfig = new org.apache.ibatis.session.Configuration();  
    myBatisConfig.setLazyLoadingEnabled(true);  
    myBatisConfig.setAggressiveLazyLoading(false);  
    myBatisConfig.setCacheEnabled(true);  
    myBatisConfig.addMappers("com.oc.greenbean.mybatis.mapper");  
    sqlSessionFactoryBean.setConfiguration(myBatisConfig);  
    return sqlSessionFactoryBean.getObject();  
}
```

## 配置mapper

```java
package com.oc.greenbean.mybatis.mapper;

public interface UserMapper {  
    User getUserById(Integer id);  
  
    User getUserByUsername(String username);  
  
    void insertUserBasicInfo(User user);  
  
    void insertUserAuthority(@Param("userId")Integer userId, @Param("authority")List<String> authority);  
  
    void updateNickname(@Param("username")String username, @Param("nickname")String nickname);  
  
    void updateAvatar(@Param("username")String username, @Param("avatar") String avatar);  
}
```

`src/main/resources/com/oc/greenbean/mybatis/mapper/UserMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<!--suppress SqlDialectInspection, SqlNoDataSourceInspection -->  
<mapper namespace="com.oc.greenbean.mybatis.mapper.UserMapper">  
  <select id="getUserById" resultType="com.oc.greenbean.domain.User">  
    select * from t_user where id = #{id}  
  </select>  
  
  <select id="getUserByUsername" resultType="com.oc.greenbean.domain.User">  
    select * from t_user where username = #{username}  
  </select>  
  
  <insert id="insertUserBasicInfo">  
    <selectKey keyProperty="id" order="AFTER" resultType="int">select last_insert_id()</selectKey>  
    insert into t_user (username, password, enabled, nickname) values(#{username}, #{password}, #{enabled}, #{nickname})  
  </insert>  
  
  <insert id="insertUserAuthority">  
    insert into t_authority (user_id, authority) values  
    <foreach item='item' collection='authority' separator=','>  
      (#{userId}, #{item})  
    </foreach>  
  </insert>  
  
  <update id="updateNickname">  
    update t_user set nickname = #{nickname} where username = #{username}  
  </update>  
  
  <update id="updateAvatar">  
    update t_user set avatar = #{avatar} where username = #{username}  
  </update>  
</mapper>
```

## 注册mapper

```java
@Bean  
public UserMapper userMapper(SqlSessionFactory sqlSessionFactory) {  
    SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory);  
    return sqlSessionTemplate.getMapper(UserMapper.class);  
}
```