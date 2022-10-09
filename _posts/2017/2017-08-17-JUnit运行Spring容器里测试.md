---
title: JUnit运行Spring容器里测试
tags: 
  - 实践案例
---

## 代码例子

### Spring项目

使用JUnit 4

<!--more-->

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = WebConfig.class)
public class RootControllerTest {
}
```

### Spring Boot项目

使用JUnit 5

```java
@SpringBootTest
class LanguageTrainerApplicationTests {
	@Test
	void contextLoads() {
	}
}
```



