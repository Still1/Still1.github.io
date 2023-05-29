---
title: Java容器ArrayList源码分析
tags: [Java, 源码分析, 概念原理]
---

本文源码基于JDK8
{:.info}

## 数据结构

`ArrayList`的底层数据结构为对象数组

```java
transient Object[] elementData;
```

## 初始化

初始化不显式指定初始容量，则默认容量为10，且数组暂不初始化，暂时使用共享空数组`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，等到有元素加入才初始化。如显式指定初始容量，且容量不为0，则马上按初始容量初始化。显式指定初始容量为0，则使用共享空数组`EMPTY_ELEMENTDATA`

```java
    // 初始容量
    private static final int DEFAULT_CAPACITY = 10;

    // 显式指定长度为0，使用此共享空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // 使用默认长度，在未加入元素时，暂时使用此共享空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        // 按初始容量构造elementData对象数据
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        // 显式指定长度为0，使用共享空数组EMPTY_ELEMENTDATA
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList() {
    // 不指定长度，则使用共享空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，这个空数组将在加入第一个元素时按默认长度扩展
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

## 扩容

`ArrayList`最大容量为`Integer.MAX_VALUE`，`ArrayList`最大安全容量为`Integer.MAX_VALUE - 8`，大于最大安全长度在某些版本的虚拟机环境里会导致`OutOfMemoryError`，非必要不要超出最大安全长度

使用共享空数组`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，即还没放进过元素的`ArrayList`，首次扩容至少扩到`DEFAULT_CAPACITY`，默认为10

当要求的容量大于`ArrayList`当前容量，则会调用`grow`方法扩容，先进行一次常规扩容，常规扩容后的容量大概为原始容量的1.5倍，若常规扩容后还不满足要求的容量，则直接按照要求的容量大小来扩容

若最终确认扩容的容量大小大于`ArrayList`最大安全容量，且还没超出最大容量，则直接扩容到最大容量

```java
// 最大安全容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

扩容的核心方法`ensureExplicitCapacity`和`grow`

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 当前要求的最小容量比elementData长度要大，需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 常规扩充方法
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        // 还不够的话按要求的最小量来扩充
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        // 扩容的容量大小大于最大安全容量
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

处理大容量的方法`hugeCapacity`

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        // 超出最大容量
        throw new OutOfMemoryError();
    // 扩容大小大于最大安全容量，且还没超出最大容量，则直接扩容到最大容量
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

## 增加元素

```java
public boolean add(E e) {
    // 要求当前元素数量加1的容量大小，如容量大小不足则触发扩容
    ensureCapacityInternal(size + 1);
    // 新元素放到数组尾部
    elementData[size++] = e;
    return true;
}
```

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 假如elementData是DEFAULTCAPACITY_EMPTY_ELEMENTDATA空数组，则至少扩容到DEFAULT_CAPACITY
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

## 获取元素

```java
public E get(int index) {
    // 检查index是否越界
    rangeCheck(index);
    // 按index在数组中访问对应元素
    return elementData(index);
}
```

## 提前扩容

大批量循环加入数据到`ArrayList`，可以先调用`ensureCapacity`提前进行一次大的扩容，可以提高性能，避免在循环过程中需要多次扩容

```java
public void ensureCapacity(int minCapacity) {
    // 假如elementData是DEFAULTCAPACITY_EMPTY_ELEMENTDATA空数组，则至少扩容到DEFAULT_CAPACITY
    // minCapacity小于或等于DEFAULT_CAPACITY无需预先扩容
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        ? 0
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```