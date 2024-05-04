---
title: Java Unsafe类主要功能
tags: [Java, 概念原理]
---

## 简介

通过Java中的`Unsafe`类可以执行很多较底层的操作，如直接分配堆外内存等，因此使用起来具有一定的风险，`Unsafe`类的名称由此得来。`Unsafe`类通常只提供给由`BootstrapClassLoader`类加载器所加载的类使用，且在JDK源码中有较广泛的使用

JDK9及以后，除了原来的`sun.misc.Unsafe`类之外，还多出了一个`jdk.internal.misc.Unsafe`类，新多出来的这个`Unsafe`类仅供JDK內部使用，并且功能比原来的`Unsafe`类更加强大。原来的`Unsafe`类的某些方法的实现也改为了依赖新的`Unsafe`类

## 使用Unsafe

一般不建议在应用代码中直接使用`Unsafe`，若要使用，可通过反射取得`Unsafe`对象，又或者把使用`Unsafe`的应用类通过`BootstrapClassLoader`来加载

反射获得`Unsafe`对象

```java
Field field = Unsafe.class.getDeclaredField("theUnsafe");  
field.setAccessible(true);  
Unsafe unsafe = (Unsafe) field.get(null);
```

应用类通过`BootstrapClassLoader`加载，通过在启动JVM时添加参数

```shell
# ${path}代表应用类路径
java -Xbootclasspath/a: ${path}
```

## 内存操作

`Unsafe`的内存操作地址有两种寻址方式，分别为绝对地址和相对对象引用寻址，并有相应的重载方法

### 绝对地址寻址

```java
// 分配内存空间4B
long address = unsafe.allocateMemory(4);
// 绝对地址，对每个字节设置对应的值
unsafe.setMemory(address, 1, (byte)0b00110011);  
unsafe.setMemory(address + 1, 1, (byte)0b11000011);  
unsafe.setMemory(address + 2, 1, (byte)0b10011001);  
unsafe.setMemory(address + 3, 1, (byte)0b11110000);  
int anInt = unsafe.getInt(address);  
System.out.println(Integer.toBinaryString(anInt));  
// 释放内存
unsafe.freeMemory(address);
```

输出结果

```
11110000100110011100001100110011
```

### 对象引用寻址

```java
private static class Wrapper {  
    private int value;  
}
```

```java
Wrapper wrapper = new Wrapper();  
System.out.println(wrapper.value);  
  
// 计算对象属性的地址偏移量
long offset = unsafe.objectFieldOffset(Wrapper.class.getDeclaredField("value"));  
// 修改属性值
unsafe.putInt(wrapper, offset, 2);  
  
System.out.println(wrapper.value);
```

输出结果

```
0
2
```

## 内存屏障

### 读屏障

读屏障，读屏障前的读操作不能被重排序到读屏障后，读屏障后的读操作和写操作不能被重排序到读屏障前。CPU执行读屏障指令时，会处理invalidate queue，将该失效的缓存行设置状态为invalidate。然后再执行读屏障指令后面的操作

对应LoadLoad屏障和LoadStore屏障

```java
unsafe.loadFence();
```

### 写屏障

写屏障，写屏障前的写操作和读操作不能被重排序到写屏障后，写屏障后的写操作不能被重排序到写屏障前。CPU执行写屏障指令时，会处理store buffer，将当前存在于store buffer的数据打上记号，在这些打上记号的数据被写入cache对应的缓存行之前，写屏障后的写操作只能存放于store buffer，无法被写入cache

对应StoreStore屏障和LoadStore屏障

```java
unsafe.storeFence();
```

### 全能屏障

全能屏障，兼具读屏障与写屏障的功能

对应LoadLoad屏障、StoreStore屏障、LoadStore屏障和StoreLoad屏障

```java
unsafe.fullFence();
```

## CAS

CAS全称Compare And Swap，将内存中某个位置的值与预期值比较，若相等，则替换为新的值，返回`true`，若不相等，则不替换，返回`false`。CAS由CPU提供操作的原子性

`Unsafe`的CAS操作有两种寻址方式，分别为绝对地址和相对对象引用寻址。使用绝对地址时简单地把第一个参数设置为`null`即可

```java
long address = unsafe.allocateMemory(4);  
unsafe.setMemory(address, 1, (byte)0b00110011);  
unsafe.setMemory(address + 1, 1, (byte)0b11000011);  
unsafe.setMemory(address + 2, 1, (byte)0b10011001);  
unsafe.setMemory(address + 3, 1, (byte)0b11110000);  
int anInt = unsafe.getInt(address);  
System.out.println(Integer.toBinaryString(anInt));  
unsafe.compareAndSwapInt(null, address, 0b11110000100110011100001100110011, 0b0);  
int afterCAS = unsafe.getInt(address);  
System.out.println(Integer.toBinaryString(afterCAS));  
unsafe.freeMemory(address);
```

输出结果

```
11110000100110011100001100110011
0
```

JDK 11下的`compareAndSwapInt`方法源码

```java
/**  
 * Atomically updates Java variable to {@code x} if it is currently
 * holding {@code expected}
 *
 * <p>This operation has memory semantics of a {@code volatile} read
 * and write.  Corresponds to C11 atomic_compare_exchange_strong
 *
 * @return {@code true} if successful
 */
@ForceInline  
public final boolean compareAndSwapInt(Object o, long offset,  
                                       int expected,  
                                       int x) {  
    return theInternalUnsafe.compareAndSetInt(o, offset, expected, x);  
}

@HotSpotIntrinsicCandidate  
public final native boolean compareAndSetInt(Object o, long offset,  
                                             int expected,  
                                             int x);
```

`Unsafe`类提供的CAS操作，读写都自带volatile语义，可保证相关变量的可见性
{:.info}

## 线程调度

阻塞当前线程，直到被调用相应的`unpark`，或被中断，或超时

```java
unsafe.park(false, 0);
```

唤醒线程

```java
unsafe.unpark(thread);
```