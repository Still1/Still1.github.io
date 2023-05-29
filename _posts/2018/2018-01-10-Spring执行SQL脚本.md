---
title: Spring执行SQL脚本
tags: [软件测试, Java, Spring]
---

## 执行SQL脚本

```java
Connection connection = dataSource.getConnection();
Resource resource = applicationContext.getResource("classpath:/sql/setup/t_corpus.sql");
EncodedResource encodedResource = new EncodedResource(resource, StandardCharsets.UTF_8);
ScriptUtils.executeSqlScript(connection, encodedResource);
connection.close();
```