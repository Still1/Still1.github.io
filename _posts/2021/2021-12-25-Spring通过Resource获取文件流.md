---
title: Spring通过Resource获取文件流
tags: [Spring]
---

## 代码实例

文件路径可基于classpath，也可以使用文件系统的绝对路径

```java
Resource resource = applicationContext.getResource("classpath:/redisson.yml");  
InputStream inputStream = resource.getInputStream();
```