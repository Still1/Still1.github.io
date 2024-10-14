---
title: Spring bean构造后、销毁前的钩子处理函数
tags: [Spring]
---

## bean初始化过程

![https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071444360.png](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071444360.png)

## 代码实例

```java
// 声明bean时定义initMethod和destroyMethod
@Bean(initMethod = "initMethod", destroyMethod = "destroyMethod")  
public SampleBean sampleBean() {  
    return new SampleBean();  
}
```

```java
package com.tinktek.odm.manager.impl;  
  
import org.springframework.beans.factory.DisposableBean;  
import org.springframework.beans.factory.InitializingBean;  
  
import javax.annotation.PostConstruct;  
import javax.annotation.PreDestroy;  
  
public class SampleBean implements InitializingBean, DisposableBean {  

    // 构造后钩子函数，执行顺序从上到下
    @PostConstruct  
    public void postConstruct() {  
        System.out.println("1");  
    }  
  
    @Override  
    public void afterPropertiesSet() throws Exception {  
        System.out.println("2");  
    }  
  
    public void initMethod() {  
        System.out.println("3");  
    }  

    // 销毁前钩子函数，执行顺序从上到下
    @PreDestroy  
    public void preDestroy() {  
        System.out.println("4");  
    }  
  
    @Override  
    public void destroy() throws Exception {  
        System.out.println("5");  
    }  
  
    public void destroyMethod() {  
        System.out.println("6");  
    }  
}
```