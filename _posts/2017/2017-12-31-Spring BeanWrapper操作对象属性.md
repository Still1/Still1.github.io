---
title: Spring BeanWrapper操作对象属性
tags: [Spring]
---

## 代码实例

```java
UserCorpusStat userCorpusStat = new UserCorpusStat(userId, corpusId);
BeanWrapper userCorpusStatBeanWrapper = new BeanWrapperImpl(userCorpusStat);

Integer count = (Integer)userCorpusStatBeanWrapper.getPropertyValue(userCorpusStatCountField);
userCorpusStatBeanWrapper.setPropertyValue(userCorpusStatCountField, 1);
```