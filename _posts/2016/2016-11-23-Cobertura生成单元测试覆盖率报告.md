---
title: Cobertura生成单元测试覆盖率报告
tags: [Cobertura]
---

## 配置Maven插件

```xml
<plugin>  
  <groupId>org.codehaus.mojo</groupId>  
  <artifactId>cobertura-maven-plugin</artifactId>  
  <version>2.7</version>  
  <executions>    
    <execution>
      <id>default-site</id>  
      <phase>site</phase>  
      <goals>
        <goal>clean</goal>  
        <goal>cobertura</goal>  
      </goals>
    </execution>
  </executions>
</plugin>
```

## 运行Cobertura

### 手动运行

```shell
mvn cobertura:cobertura
```

### Maven自动运行

通过`<execution>`标签将Cobertura绑定到Maven的site阶段

```shell
mvn clean package site
```