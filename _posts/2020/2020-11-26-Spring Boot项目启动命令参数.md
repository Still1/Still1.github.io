---
title: Spring Boot项目启动命令参数
tags: [Spring Boot]
---

## 启动命令

基础启动命令

```shell
java -jar /opt/languageTrainer/LanguageTrainer.jar
```

指定Spring Boot profile

```shell
java -jar /opt/languageTrainer/LanguageTrainer.jar --spring.profiles.active=prod
```

指定系统属性

```shell
java -Dfile.encoding=UTF-8 -jar /opt/languageTrainer/LanguageTrainer.jar
```

指定非标准选项

```shell
java -jar -Xmx512m -Xms256m /opt/languageTrainer/LanguageTrainer.jar
```

不间断运行

```shell
nohup java -jar /opt/languageTrainer/LanguageTrainer.jar > /opt/logs/temp.txt 2>&1 &
```