---
title: MyBatis-Plus代码生成器使用
tags: [MyBatis-Plus]
---

## Maven依赖

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-generator</artifactId>  
    <version>3.5.3</version>  
</dependency>  
  
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-boot-starter</artifactId>  
    <version>3.5.3</version>  
</dependency>  
  
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>org.postgresql</groupId>  
    <artifactId>postgresql</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-freemarker</artifactId>  
</dependency>  
  
<dependency>  
    <groupId>org.jetbrains</groupId>  
    <artifactId>annotations</artifactId>  
    <version>24.0.1</version>  
</dependency>
```

## 配置数据源

```yml
spring:  
  datasource:  
    username: odm  
    password: odm
    url: jdbc:postgresql://110.41.3.137:54321/odmdb?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=Asia/Shanghai  
    driver-class-name: org.postgresql.Driver
```

## 启动类调用代码生成

```java
package com.oliverclio.code;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.context.ConfigurableApplicationContext;  
  
@SpringBootApplication  
public class CodeGeneratorApplication {  
  
    public static void main(String[] args) {  
        ConfigurableApplicationContext applicationContext = SpringApplication.run(CodeGeneratorApplication.class, args);  
        CodeGenerator codeGenerator = applicationContext.getBean(CodeGenerator.class);  
        codeGenerator.generate();  
    }  
}
```

## 代码生成配置

```java
package com.oliverclio.code;  
  
import com.baomidou.mybatisplus.generator.FastAutoGenerator;  
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;  
import com.baomidou.mybatisplus.generator.config.TemplateType;  
import com.baomidou.mybatisplus.generator.config.builder.CustomFile;  
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;  
import lombok.AllArgsConstructor;  
import org.springframework.stereotype.Component;  
  
import javax.sql.DataSource;  
import java.util.ArrayList;  
import java.util.List;  
  
@AllArgsConstructor  
@Component  
public class CodeGenerator {  
    public static final String[] TABLE_ARRAY = {"t_user"};  
    public static final String TABLE_PREFIX = "t_";  
    public static final String PACKAGE_NAME = "com.oliverclio";  
    public static final String MODULE_NAME = "odm";  
    public static final String OUTPUT_DIR = "D:/generator/";  
    public static final String[] IGNORE_COLUMNS = {"id", "created_by", "created_time", "updated_by", "updated_time", "deleted"};  
    public static final String SERVICE_FILE_NAME = "%sService";  
  
    public static final List<CustomFile> CUSTOM_FILES = new ArrayList<CustomFile>() {  
        {  
            this.add(new CustomFile.Builder().fileName("Manager.java").templatePath("/templates/manager.java.ftl").packageName("manager").build());  
            this.add(new CustomFile.Builder().fileName("ManagerImpl.java").templatePath("/templates/managerImpl.java.ftl").packageName("manager.impl").build());  
            this.add(new CustomFile.Builder().fileName("Condition.java").templatePath("/templates/condition.java.ftl").packageName("dto.condition").build());  
        }  
    };  
  
    private final DataSource dataSource;  
  
    public void generate() {  
        FastAutoGenerator.create(new DataSourceConfig.Builder(dataSource))  
                .globalConfig(builder -> builder.outputDir(OUTPUT_DIR))  
                .packageConfig(builder -> builder.parent(PACKAGE_NAME)  
                        .moduleName(MODULE_NAME))  
                .strategyConfig(builder -> builder.addInclude(TABLE_ARRAY).addTablePrefix(TABLE_PREFIX)  
                        .entityBuilder().addIgnoreColumns(IGNORE_COLUMNS)  
                        .serviceBuilder().formatServiceFileName(SERVICE_FILE_NAME))  
                .templateConfig(builder -> builder.disable(TemplateType.CONTROLLER))  
                .injectionConfig(builder -> builder.customFile(CUSTOM_FILES))  
                .templateEngine(new FreemarkerTemplateEngine())  
                .execute();  
    }  
}
```

## 自定义模板

默认模板文件夹路径 `src/main/resources/templates/`

`condition.java.ftl`
```
package ${package.Parent}.dto.condition;  
  
import io.swagger.annotations.ApiModel;  
import lombok.Data;  
import lombok.EqualsAndHashCode;  
  
import java.io.Serializable;  
  
/**  
 * ${table.comment!}查询条件  
 */  
@Data  
@EqualsAndHashCode(callSuper = false)  
@ApiModel(description = "${table.comment!}查询条件")  
public class ${entity}Condition extends DmsCommonCondition {  
}
```

`entity.java.ftl`
```
package ${package.Entity};  
  
import com.oliverclio.dms.permission.OrganizationIdOwner;  
import io.swagger.annotations.ApiModel;  
import io.swagger.annotations.ApiModelProperty;  
import lombok.Data;  
import lombok.EqualsAndHashCode;  
  
import java.io.Serializable;  
  
/**  
 * ${table.comment!}  
 */  
@Data  
@EqualsAndHashCode(callSuper = false)  
@ApiModel(description = "${table.comment!}")  
public class ${entity} extends DmsCommonEntity implements Serializable, OrganizationIdOwner {  
<#-- ----------  BEGIN 字段循环遍历  ---------->
<#list table.fields as field>  
    <#if field.comment!?length gt 0>  
    /**  
     * ${field.comment}  
     */  
    @ApiModelProperty("${field.comment}")  
    </#if>  
    private ${field.propertyType} ${field.propertyName};  
</#list>  
<#------------  END 字段循环遍历  ---------->}
```

`manager.java.ftl`
```
package ${package.Parent}.manager;  
  
import ${package.Parent}.dto.condition.${entity}Condition;  
import ${package.Entity}.${entity};  
import ${package.Mapper}.${table.mapperName};  
  
/**  
 * ${table.comment!}通用处理接口  
 */  
public interface ${entity}Manager extends DmsCommonManager<${entity}, ${entity}Condition, ${table.mapperName}> {  
}
```

`managerImpl.java.ftl`
```
package ${package.Parent}.manager.impl;  
  
import ${package.Parent}.dto.condition.${entity}Condition;  
import ${package.Entity}.${entity};  
import ${package.Mapper}.${table.mapperName};  
import com.oliverclio.dms.manager.AbstractDmsCommonManager;  
import org.springframework.stereotype.Component;  
  
/**  
 * ${table.comment!}通用处理实现类  
 */  
@Component  
public class ${entity}ManagerImpl extends AbstractDmsCommonManager<${entity}, ${entity}Condition, ${table.mapperName}> implements ${entity}Manager {  
}
```

`mapper.java.ftl`
```
package ${package.Mapper};  
  
import ${package.Entity}.${entity};  
import ${superMapperClassPackage};  
import org.springframework.stereotype.Repository;  
  
/**  
 * ${table.comment!}Mapper  
 */  
@Repository  
public interface ${table.mapperName} extends ${superMapperClass}<${entity}> {  
}
```

`mapper.xml.ftl`
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<!--suppress SqlDialectInspection, SqlNoDataSourceInspection -->  
<mapper namespace="${package.Mapper}.${table.mapperName}">  
<#if enableCache>  
    <!-- 开启二级缓存 -->  
    <cache type="${cacheClassName}"/>  
  
</#if>  
<#if baseResultMap>  
    <!-- 通用查询映射结果 -->  
    <resultMap id="BaseResultMap" type="${package.Entity}.${entity}">  
<#list table.fields as field>  
<#if field.keyFlag><#--生成主键排在第一位-->  
        <id column="${field.name}" property="${field.propertyName}" />  
</#if>  
</#list>  
<#list table.commonFields as field><#--生成公共字段 -->        <result column="${field.name}" property="${field.propertyName}" />  
</#list>  
<#list table.fields as field>  
<#if !field.keyFlag><#--生成普通字段 -->        <result column="${field.name}" property="${field.propertyName}" />  
</#if>  
</#list>  
    </resultMap>  
  
</#if>  
<#if baseColumnList>  
    <!-- 通用查询结果列 -->  
    <sql id="Base_Column_List">  
<#list table.commonFields as field>  
        ${field.columnName},  
</#list>  
        ${table.fieldNames}  
    </sql>  
  
</#if>  
</mapper>
```

`service.java.ftl`
```
package ${package.Service};  
  
import ${package.Parent}.dto.condition.${entity}Condition;  
import ${package.Entity}.${entity};  
  
/**  
 * ${table.comment!}服务接口  
 */  
public interface ${table.serviceImplName} extends DmsCommonService<${entity}, ${entity}Condition> {  
}
```

`serviceImpl.java.ftl`
```
package ${package.ServiceImpl};  
  
import ${package.Parent}.dto.condition.${entity}Condition;  
import ${package.Entity}.${entity};  
import ${package.Mapper}.${table.mapperName};  
import ${package.Parent}.manager.${entity}Manager;  
import ${package.Service}.${table.serviceName};  
import org.springframework.stereotype.Service;  
  
/**  
 * ${table.comment!}服务实现类  
 */  
@Service  
public class ${table.serviceImplName} extends AbstractDmsCommonService<${entity}, ${entity}Condition, ${table.mapperName}, ${entity}Manager> implements ${table.serviceName} {  
}
```