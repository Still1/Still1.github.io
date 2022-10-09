---
title: Spring IoC容器简述
tags: 
  - 知识总结
---

## IoC

IoC，Inversion of Control，控制反转。可理解为原来要用到的bean需要自己创建，也就是说控制bean的创建过程。而引入IoC技术后，bean并不由使用者创建，创建的控制权转移到IoC容器

<!--more-->

## DI

DI，Dependency Injection，依赖注入。更强调bean之间依赖关系的处理。实际上IoC容器在创建bean期间，必然要处理bean之间的依赖关系

## IoC Service Provider

IoC容器的抽象概念，负责业务对象的构建管理和业务对象间的依赖绑定。Spring IoC容器的核心部分就是一个IoC Service Provider

![image-20220607135349560](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071353198.png)

IoC Service Provider实际功能示意图

![image-20220607140049201](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071400322.png)



## Spring IoC容器的实现

* `BeanFactory` 只实现最核心的IoC Service Provider部分。bean在需要使用时创建。
* `ApplicationContext` 以`BeanFactory`为基础，增加更多外围服务。启动即创建所有bean。

## Spring IoC容器功能实现的各个阶段

![image-20220607140323354](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071403377.png)

### 容器启动阶段

容器启动触发

1. 加载Configuration Metadata，如XML、注解等。一般需要用到`BeanDefinitionReader`
2. 将配置信息映射为`BeanDefinition`，并注册到`BeanDefinitionRegistry`
3. 调用已注册到容器中的`BeanFactoryPostProcessor`，对`BeanDefinition`做额外的操作。例如`PropertyPlaceholderConfigurer`处理`BeanDefinition`中的占位符。又如`CustomEditorConfigurer`注册更多的Bean到容器中，相当于增加了`BeanDefinition`

### Bean实例化阶段

第一次真正获取容器中的某个bean时触发

![image-20220607144452907](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206071444360.png)

1. 选定实例化bean策略实现。例如`CglibSubclassingInstantiationStrategy`通过CGLIB生成目标类的子类，并使用反射实例化对象。
2. 根据实例化bean策略和`BeanDefinition`，创建对应的bean，并包装为`BeanWrapper`
3. 通过`BeanWrapper`辅助设置对象属性
4. `Aware`接口依赖注入处理。如实现了`ApplicationContextAware`，会将`ApplicationContext`实例注入到bean中。实际上是通过`BeanPostProcessor`实现的，调用了`ApplicationContextAwareProcessor`的`postProcessBeforeInitialization`方法
5. 调用已注册到容器中的`BeanPostProcessor`的`postProcessBeforeInitialization`方法，对bean作额外的操作
6. 如果bean实现了`InitializingBean`接口，则调用`afterPropertiesSet`方法，对bean作额外的操作
7. 如果bean配置了init-method，则调用该方法。这相当于低侵入性的步骤6
8. 调用已注册到容器中的`BeanPostProcessor`的`postProcessAfterInitialization`方法，对bean作额外的操作
9. 如果bean实现了`DisposableBean`接口，则注册一个用于对象销毁的回调，回调调用bean实现接口的`destroy`方法
10. 如果bean配置了destroy-method，则注册一个用于对象销毁的回调，回调调用该方法。这相当于低侵入性的步骤9

