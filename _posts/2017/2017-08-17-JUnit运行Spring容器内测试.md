---
title: JUnit运行Spring容器内测试
tags: [JUnit]
---

## 代码例子

### Spring项目

使用JUnit 4

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