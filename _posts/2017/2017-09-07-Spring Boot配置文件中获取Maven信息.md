---
title: Spring Boot配置文件中获取Maven信息
tags: [Spring Boot, Maven]
---

## 代码实例

`application.yml`
```yml
odm:  
  system:  
    version: '@project.version@'  
    chinese-name: '@project.description@'
```

可获取Maven的pom.xml文件中的配置项

当yml配置文件中出现中文时，配置文件的字符编码配置不当会导致项目启动报错，最好都统一为UTF-8
{:.warning}