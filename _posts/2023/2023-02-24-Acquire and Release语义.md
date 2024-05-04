---
title: Acquire and Release语义
tags: [概念原理]
---

## 概述

Acquire and Release语义表示多线程并发访问共享内存的一种方式，表现为线程间的一种合作关系，在单个写线程的场景下使用。写线程的写操作完成后，发布一个操作完成的信号，各个读线程接收到该信号后，对值进行读取，实现线程间信息的可靠传输

## 语义定义

一个对共享内存的读操作（也可以是read-modify-write操作）可以具有acquire语义，具有acquire语义的读操作被称为read-acquire。Acquire语义消除了read-acquire与后续的读取与写入操作的内存重排序

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230412054929.png" width="300px" />

一个对共享内存的写操作（也可以是read-modify-write操作）可以具有release语义，具有release语义的写操作被称为write-release。Release语义消除了write-release与前置的读取与写入操作的内存重排序

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230412054950.png" width="300px" />

## 内存屏障实现Acquire and Release语义

* acquire语义可以通过在read-acquire后面加入LoadLoad屏障和LoadStore屏障实现
* release语义可以通过在write-release前面加入StoreStore屏障和LoadStore屏障实现

屏障的实现实际上比语义的定义限制更为严格，因此两者并不完全等同
{:.info}

## Acquire and Release语义实现线程间信息传递

需要满足以下条件
* read-acquire和write-release都需要是一个原子操作
* read-acquire和write-release操作的是同一个变量

根据内存屏障实现来理解，有以下示意图

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230413074019.png" width="700px" />

payload表示线程1与线程2需要传递的信息，guard表示payload是否已准备好的信号。read-acquire和write-release都是一个原子操作，且操作的是同一个变量guard。write-release前面的StoreStore屏障保证了线程1中对payload的写入比guard的写入更早完成，read-acquire后面的LoadLoad屏障保证了若线程2读取到了guard的更新，则后续一定能读取到payload的更新。此过程不保证线程2什么时候能读取到guard的更新

实现线程间信息传递，并不依赖LoadStore屏障
{:.info} 

类Java伪代码实现的一个例子如下图

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230413074227.png" width="800px" />

`data`作为payload的角色，`ready`作为guard的角色。可以保证线程2中，`myReady`的值为`true`时，`myData`的值为`1`

Java中，对基础变量的简单读写(plain loads and plain stores)是原子操作。上例中，需要确保对ready的读写是原子操作
{:.info}

Java中，`Unsafe`类提供的内存屏障操作，`storeFence()`对应StoreStore屏障和LoadStore屏障，`loadFence()`对应LoadLoad屏障和LoadStore屏障
{:.info}

## Acquire and Release语义实现互斥锁

实现线程间信息传递是实现互斥锁的基础

根据内存屏障实现来理解，有以下示意图

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230414110240.png" width="700px" />

互斥锁的实现依赖LoadStore屏障，将所有的内存读写操作限制在read-acquire和write-release之间，针对互斥锁的实现来说就是限制在持有锁期间。互斥锁mutex充当了原来guard的作用，作为线程间传递信息的信号。这样的实现可以保证，当线程2获取锁后，线程1在持有锁期间对内存所做的任何写操作，对线程2都是可见的

实现互斥锁时，一个线程获得锁或释放锁后，其它线程必须能够及时观测到。由于Acquire and Release语义不保证线程2读取到`mutex`变量更新的时间，所以只依赖Acquire and Release语义并不足够，还需要在write-release之后和read-acquire之前加入StoreLoad屏障。write-release之后的StoreLoad屏障保证通过该屏障时，`mutex`变量的内存写入操作已完成，read-acquire之前的StoreLoad屏障保证该屏障后读取到`mutex`变量的更新

`mutex`变量的读写满足volatile语义，屏障的实现实际上比语义的定义限制更为严格，因此两者并不完全等同
{:.info}

所有线程观测到的对`mutex`变量的内存写入顺序是一致的，且此顺序与程序源代码顺序一致，我们称此为顺序一致性(Sequential Consistency)
{:.info}

类Java伪代码实现的一个例子如下图

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230414120428.png" width="800px" />

`mutex`的值为0时表示互斥锁未被持有，`mutex`的值为1时表示互斥锁被持有。线程2尝试CAS操作成功之后，线程持有互斥锁并读取线程1对内存所做的修改，最后释放锁

CAS操作属于原子read-modify-write操作
{:.info}

Java中，CAS操作伪代码`cas(mutex, 0, 1)`可通过`Unsafe`类的`compareAndSwapInt`方法实现
{:.info}

Java中，`Unsafe`类提供的内存屏障操作，`fullFence()`对应全能屏障(full memory fence)，同时兼具LoadLoad、StoreStore、LoadStore、StoreLoad屏障的作用
{:.info}

## 参考文档

* [https://preshing.com/20120913/acquire-and-release-semantics/](https://preshing.com/20120913/acquire-and-release-semantics/)