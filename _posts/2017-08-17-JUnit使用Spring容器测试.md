---
title: JUnit使用Spring容器测试
tags: 
  - JUnit
  - Spring
modify_date: 2017-08-17
---

## 代码例子

<!--more-->

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = WebConfig.class)
public class RootControllerTest {
}
```

