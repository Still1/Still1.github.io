---
title: Java Concurrency In Practice线程安全相关注解使用
tags: [Java]
---

## 代码实例

```xml
<dependency>  
    <groupId>net.jcip</groupId>  
    <artifactId>jcip-annotations</artifactId>  
    <version>1.0</version>  
    <scope>provided</scope>  
</dependency>
```

* `@ThreadSafe`标注线程安全类
* `@NotThreadSafe`标注非线程安全类
* `@GuardedBy("lock")`标注状态被哪个锁保护
* `@Immutable`标注不可变状态