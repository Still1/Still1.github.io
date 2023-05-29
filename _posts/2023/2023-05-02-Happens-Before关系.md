---
title: Happens-Before关系
tags: [概念原理, 多线程]
---

## 定义

假设有操作A和操作B，且有A happens-before B的关系，则表明操作A对内存产生的效果在操作B被执行前，对操作B等效可见

以下两个条件满足任意一个即可视为等效可见
* 操作A对内存产生的效果确实对操作B可见
* 操作A对内存产生的效果不对操作B产生任何影响

## 确立关系

确立happens-before关系的概览图如下，图中只列出一些常见的确立关系方式

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230414145627.png" width="700px" />

### Program Order

同一线程执行操作A和操作B，若代码顺序操作A在操作B前面，则有A happens-before B的关系

### Synchronizes-with

Synchronizes-with关系是一种线程之间安全传输数据的方式，某一线程首先独自向内存写入数据，待写入完成后，发布一个操作完成的信号，其它线程接收到该信号后，对值进行读取

Synchronizes-with关系中有两个关键元素，分别为payload和guard variable。其中payload表示线程之间需要传输的数据，而guard variable表示payload写入是否已操作完成的信息

[Acquire and release语义](https://blog.oliverclio.com/2023/02/24/Acquire-and-Release%E8%AF%AD%E4%B9%89.html#acquire-and-release%E8%AF%AD%E4%B9%89%E5%AE%9E%E7%8E%B0%E7%BA%BF%E7%A8%8B%E9%97%B4%E4%BF%A1%E6%81%AF%E4%BC%A0%E9%80%92)是synchronizes-with关系的一种实现，如下图所示

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230414161115.png" width="700px" />

一旦线程2中读取到guard的更新，线程1的write-release和线程2的read-acquire的synchronizes-with关系确立，同时确立线程1的write-release之前的操作happens-before线程2的read-acquire之后的操作，因此线程1的write-release和线程2的read-acquire又被称为happens-before edge


Synchronizes-with关系是一种运行时关系，而非程序代码之间的关系。如上例中，假如线程2中的read-acquire早一点被执行，没有读取到guard的更新，则不存在synchronizes-with关系，如下图
{:.warning}

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230415023433.png" width="700px" />

## 具体实现

Happens-before关系的具体实现由各编程语言决定。编程语言在描述自己的软件内存模型时，会对其中存在的happens-before关系作出描述，同时保证这些happens-before关系必定成立。开发人员一般只需要关注具体的编程语言的规范中描述了哪些happens-before关系并加以利用即可

Java规范中happens-before规则包括
* 程序顺序规则，同一线程执行操作A和操作B，若代码顺序操作A在操作B前面，则有A happens-before B的关系
* 监视器锁规则，锁释放前的操作happens-before同一个锁再次被持有后的操作
* volatile变量规则，对volatile变量写入前的操作happens-before后续同一个volatile变量被读取后的操作
* 线程启动规则，调用`thread.start()`前的操作happens-before新线程中的操作
* 线程结束规则，线程中的操作happens-before其它线程检测到该线程已经结束之后的操作，检测到线程结束包括从`thread.join()`方法返回，或`thread.isAlive()`返回`false`
* 中断规则，调用`thread.interrupt()`前的操作happens-before被中断的线程检测到中断后的操作，检测到中断包括抛出`InterruptedException`，或调用`isInterrupted()`，或调用`interrupted()`
* 终结器规则，对象构造函数中的操作happens-before对象终结器（`finalize`方法）中的操作
* 传递性，有操作A、操作B和操作C，假如A happens-before B，且B happens-before C，则有A happens-before C
* 任何对象的默认初始化操作happens-before该对象的其它操作（除了default-writes）

## 参考文档

* [https://preshing.com/20130702/the-happens-before-relation/](https://preshing.com/20130702/the-happens-before-relation/)
* [https://preshing.com/20130823/the-synchronizes-with-relation/](https://preshing.com/20130823/the-synchronizes-with-relation/)
* Brain Goetz等，Java并发编程实战(Java Concurrency in Practice)，机械工业出版社，2012：280
* https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.5