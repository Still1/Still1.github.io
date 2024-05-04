---
title: MyBatis使用SQL查询
tags: [MyBatis]
---

## 定义Mapper接口

```java
@Repository  
public interface UserLoginLogMapper {  
    Long getLoginLogCountByLoginTime(@Param("userId")Integer userId, @Param("startTime") LocalDateTime startTime, @Param("endTime") LocalDateTime endTime);  
}
```

## 编写SQL

XML文件默认放在与Mapper接口同路径下

如Mapper接口在`com.oliverclio.languagetrainer.mapper`包下，则XML文件放在路径`/resources/com/oliverclio/languagetrainer/mapper`下

`UserLoginLogMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "https://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.oliverclio.languagetrainer.mapper.UserLoginLogMapper">  
    <select id="getLoginLogCountByLoginTime" resultType="java.lang.Long">  
        select count(*) from t_user_login_log  
        where user_id = #{userId}  
        and login_time &gt;= #{startTime}  
        and login_time &lt; #{endTime}  
    </select>  
</mapper>
```