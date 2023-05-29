---
title: Java容器HashMap源码分析
tags: [Java, 源码分析, 概念原理]
mathjax: true
---

本文源码基于JDK8
{:.info}

## 数据结构

`HashMap`的基础数据结构为一个链表数组

```java
transient Node<K,V>[] table;
```

内部类`Node`代表链表节点，同时实现了接口`Map.Entry`，链表的一个节点也就是一个键值对

```java
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    V value;  
    Node<K,V> next;  
  
    Node(int hash, K key, V value, Node<K,V> next) {  
        this.hash = hash;  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }  
  
    public final K getKey()        { return key; }  
    public final V getValue()      { return value; }  
    public final String toString() { return key + "=" + value; }  
  
    public final int hashCode() {  
        return Objects.hashCode(key) ^ Objects.hashCode(value);  
    }  
  
    public final V setValue(V newValue) {  
        V oldValue = value;  
        value = newValue;  
        return oldValue;  
    }  
  
    public final boolean equals(Object o) {  
        if (o == this)  
            return true;  
        if (o instanceof Map.Entry) {  
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;  
            if (Objects.equals(key, e.getKey()) &&  
                Objects.equals(value, e.getValue()))  
                return true;  
        }  
        return false;  
    }  
}
```

默认情况下，`HashMap`加入一个新元素后，一个链表元素个数大于8，且容器大小大于等于64时，链表将转化为红黑树

```java
static final int TREEIFY_THRESHOLD = 8;
static final int MIN_TREEIFY_CAPACITY = 64;
```

链表转化为红黑树

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {  
    int n, index; Node<K,V> e;  
    // 容器大小小于转化限制时，不转化红黑树，而是先扩容
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)  
        resize();  
    else if ((e = tab[index = (n - 1) & hash]) != null) {  
        TreeNode<K,V> hd = null, tl = null;  
        do {  
            TreeNode<K,V> p = replacementTreeNode(e, null);  
            if (tl == null)  
                hd = p;  
            else {  
                p.prev = tl;  
                tl.next = p;  
            }  
            tl = p;  
        } while ((e = e.next) != null);  
        if ((tab[index] = hd) != null)  
            hd.treeify(tab);  
    }  
}
```

默认情况下，在`HashMap`重新调整容器大小时（调用`resize`方法），假如红黑树节点数小于等于6，则将红黑树转化为链表

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

数组中的一个元素，代表一个链表或一颗红黑树，我们也可以统称其为一个桶（bin)
{:.info}

## 哈希求模优化

在通过key的哈希值定位元素应该放置在数组的哪个位置时，`HashMap`并非通过求模来运算，而是转化为按位与运算来优化性能。而转化的前提则是容量大小必须满足$2^n\ \ (n \ge 0)$，设容量大小为$l$，哈希值为$h$，满足条件后有一下等式成立

$$h \% l = h\&(l-1)$$

针对上式的简化例子如下，令容量大小$l=4$，则有如下结果



| $h$  | $h\%l$   | $h\&(l-1)$                      |
| ---- | -------- | ------------------------------- |
| $0$  | $0\%4=0$ | $(0000)_2\&(0011)_2=(0000)_2=0$ |
| $1$  | $1\%4=1$ | $(0001)_2\&(0011)_2=(0001)_2=1$ |
| $2$  | $2\%4=2$ | $(0010)_2\&(0011)_2=(0010)_2=2$ |
| $3$  | $3\%4=3$ | $(0011)_2\&(0011)_2=(0011)_2=3$ |
| $4$  | $4\%4=0$ | $(0100)_2\&(0011)_2=(0000)_2=0$ |
| $5$  | $5\%4=1$ | $(0101)_2\&(0011)_2=(0001)_2=1$ |
| $6$  | $6\%4=2$ | $(0110)_2\&(0011)_2=(0010)_2=2$ |
| $7$  | $7\%4=3$ | $(0111)_2\&(0011)_2=(0011)_2=3$ |
| $8$  | $8\%4=0$ | $(1000)_2\&(0011)_2=(0000)_2=0$ |
| $9$  | $9\%4=1$ | $(1001)_2\&(0011)_2=(0001)_2=1$ |

可以观察到由于总是满足$l=2^n\ \ (n \ge 0)$，则$l-1$的二进制数总是形如高位连续为0，低位连续为1的形式，$h\&(l-1)$相当于只保留$h$的低位部分，在不超出$l$的范围内不断循环，结果与求模运算一致

我们将在源码中多次看到以下代码`(n - 1) & hash`，即是此优化的运用

## 扩容哈希求模优化

每一次扩容，扩容后`HashMap`容量大小为原大小的2倍，原来的元素需要重新计算哈希求模以获得新的存放位置，按照哈希求模优化公式，应对每个元素作如下计算

$$h\&(2l-1)$$

源码并没有对每一个元素都作此计算，当链表中的元素只有一个时，按上式计算，当链表中的元素超过一个时，则通过$h\&l$的值是否为0，判断元素的新位置。设元素的新存放位置为$n$，有以下公式成立
$$
n=h\&(2l-1)=
\begin{cases}
h\&(l-1), &h\&l = 0\\[2ex]
h\&(l-1) + l, &h\&l \neq 0
\end{cases}
$$

在上式中，$h\&(l-1)$实际上就是元素的原位置，所以元素的新位置要么不变，要么在原位置基础上加上原容量大小。在扩容的`resize`方法中，使用上式将原链表切分成两个链表，将两个链表放到新的对应位置上

针对上式的简化例子如下，令容量大小$l=4$，则有如下结果

| $h$  | $h\&l$               | $h\&(2l-1)$          | $h\&(l-1)$                      | $h\&(l-1) + l$                       |
| ---- | -------------------- | -------------------- | ------------------------------- | ------------------------------------ |
| $0$  | $(0000)_2\&(0100)=0$ | $(0000)_2\&(0111)=0$ | $(0000)_2\&(0011)_2=(0000)_2=0$ | $(0000)_2 + (0100)_2 = (0100)_2 = 4$ |
| $1$  | $(0001)_2\&(0100)=0$ | $(0001)_2\&(0111)=1$ | $(0001)_2\&(0011)_2=(0001)_2=1$ | $(0001)_2 + (0100)_2 = (0101)_2 = 5$ |
| $2$  | $(0010)_2\&(0100)=0$ | $(0010)_2\&(0111)=2$ | $(0010)_2\&(0011)_2=(0010)_2=2$ | $(0010)_2 + (0100)_2 = (0110)_2 = 6$ |
| $3$  | $(0011)_2\&(0100)=0$ | $(0011)_2\&(0111)=3$ | $(0011)_2\&(0011)_2=(0011)_2=3$ | $(0011)_2 + (0100)_2 = (0111)_2 = 7$ |
| $4$  | $(0100)_2\&(0100)=4$ | $(0100)_2\&(0111)=4$ | $(0100)_2\&(0011)_2=(0000)_2=0$ | $(0000)_2 + (0100)_2 = (0100)_2 = 4$ |
| $5$  | $(0101)_2\&(0100)=4$ | $(0101)_2\&(0111)=5$ | $(0101)_2\&(0011)_2=(0001)_2=1$ | $(0001)_2 + (0100)_2 = (0101)_2 = 5$ |
| $6$  | $(0110)_2\&(0100)=4$ | $(0110)_2\&(0111)=6$ | $(0110)_2\&(0011)_2=(0010)_2=2$ | $(0010)_2 + (0100)_2 = (0110)_2 = 6$ |
| $7$  | $(0111)_2\&(0100)=4$ | $(0111)_2\&(0111)=7$ | $(0111)_2\&(0011)_2=(0011)_2=3$ | $(0011)_2 + (0100)_2 = (0111)_2 = 7$ |
| $8$  | $(1000)_2\&(0100)=0$ | $(1000)_2\&(0111)=0$ | $(1000)_2\&(0011)_2=(0000)_2=0$ | $(0000)_2 + (0100)_2 = (0100)_2 = 4$ |
| $9$  | $(1001)_2\&(0100)=0$ | $(1001)_2\&(0111)=1$ | $(1001)_2\&(0011)_2=(0001)_2=1$ | $(0001)_2 + (0100)_2 = (0101)_2 = 5$ |

可以观察到由于总是满足$l=2^n\ \ (n \ge 0)$，则$l-1$的二进制数总是形如高位连续为0，低位连续为1的形式。而$2l-1$则是比$l-1$多了一个低位1，假如$h$在这个位上的值为0（即满足$h\&l=0$），则实际上这个多出的低位1不影响计算结果，有$h\&(2l-1)=h\&(l-1)$。假如$h$在这个位上的值为1（即满足$h\&l \neq 0$)，则这个多出的低位1，则会令该二进制位的按位与结果从原来的0变为1，而二进制位代表的大小为$l$，则有$h\&(2l-1)=h\&(l-1) + l$

## 初始化

`HashMap`的容器容量是指数组的大小，也可理解为桶的数量

`HashMap`默认情况下初始容量为16

```java
// 指定默认容量为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

`HashMap`的容量大小必须满足$2^n\ \ (n \ge 0)$，如果在构造函数中指定容量，但指定值不符合条件，则向上取第一个符合条件的整数

```java
public HashMap(int initialCapacity, float loadFactor) {  
    // 对指定容量的非法值和特殊值的处理
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;
    // 求出满足条件的容量大小，容量大小放在threshold属性
    this.threshold = tableSizeFor(initialCapacity);  
}
```

```java
static final int tableSizeFor(int cap) {
    // 将指定容量减1后，把最高位的1之后的所有低位全部置为1
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 处理特殊值，正常值则加1
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

`tableSizeFor`计算简化例子如下，真实指定容量的参数类型是`int`，一共32位，逻辑右移最多16位。例子简化为8位的数字，逻辑右移最多4位

假设cap初始值为二进制数$(01000010)_2$

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230529235241.png" width="300px" />

`HashMap`最大容量为$2^{30}$，这是int范围内最大的符合条件的数字。int范围内最大数字为$2^{31}-1$，不符合条件

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

## 扩容

初始化的`HashMap`在未有元素的时候，数组尚未初始化，直到需要放入元素时，调用`resize`方法中初始化数组

假如放入元素后，元素总个数大于扩容阈值（容量大小乘以扩容因子），则也会调用`resize`方法扩容，扩容后容量是原容量的2倍

默认扩容因子为0.75

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

`HashMap`扩容的核心方法`resize`

```java
final Node<K,V>[] resize() {  
    Node<K,V>[] oldTab = table;  
    int oldCap = (oldTab == null) ? 0 : oldTab.length;  
    int oldThr = threshold;  
    int newCap, newThr = 0;  
    if (oldCap > 0) {  
        if (oldCap >= MAXIMUM_CAPACITY) {  
            threshold = Integer.MAX_VALUE;  
            return oldTab;  
        }
        // 常规情况下扩容，容量大小与扩容阈值都乘以2  
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
                 oldCap >= DEFAULT_INITIAL_CAPACITY)  
            newThr = oldThr << 1; // double threshold  
    }  
    else if (oldThr > 0) // initial capacity was placed in threshold  
        // 数组未初始化或未有元素，初始化HashMap时容量大小已指定并放置在threshold，容量大小取threshold值
        newCap = oldThr;  
    else {               // zero initial threshold signifies using defaults  
        // 数组未初始化或未有元素，初始化HashMap时未指定容量大小，按默认值设置容量大小和扩容阈值
        newCap = DEFAULT_INITIAL_CAPACITY;  
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
    }  
    if (newThr == 0) {  
        // 数组未初始化或未有元素，在此处指定新的扩容阈值
        float ft = (float)newCap * loadFactor;  
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?  
                  (int)ft : Integer.MAX_VALUE);  
    }  
    threshold = newThr;  
    @SuppressWarnings({"rawtypes","unchecked"})  
    // 根据容量大小初始化数组，或创建扩容后数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  
    table = newTab;  
    if (oldTab != null) {
        // 旧数组有值，需要重新安排元素位置  
        for (int j = 0; j < oldCap; ++j) {  
            Node<K,V> e;  
            if ((e = oldTab[j]) != null) {  
                oldTab[j] = null;  
                if (e.next == null)  
                    // 链表只有一个元素，通过新计算的位置转移链表
                    newTab[e.hash & (newCap - 1)] = e;  
                else if (e instanceof TreeNode)
                    // 红黑树处理
                    // split方法中，假如红黑树节点数量减少到小于等于限制值，则红黑树将转化为链表  
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
                else { // preserve order  
                    Node<K,V> loHead = null, loTail = null;  
                    Node<K,V> hiHead = null, hiTail = null;  
                    Node<K,V> next;  
                    do {  
                        // 遍历原链表中的每一个元素，通过扩容哈希求模优化切分成两个链表，再放到对应的新位置上
                        next = e.next;  
                        if ((e.hash & oldCap) == 0) {  
                            if (loTail == null)  
                                loHead = e;  
                            else
                                loTail.next = e;  
                            loTail = e;  
                        }  
                        else {  
                            if (hiTail == null)  
                                hiHead = e;  
                            else
                                hiTail.next = e;  
                            hiTail = e;  
                        }  
                    } while ((e = next) != null);  
                    if (loTail != null) {  
                        loTail.next = null;  
                        newTab[j] = loHead;  
                    }  
                    if (hiTail != null) {  
                        hiTail.next = null;  
                        newTab[j + oldCap] = hiHead;  
                    }  
                }  
            }  
        }  
    }  
    return newTab;  
}
```

## 放入元素

向`HashMap`放入元素的核心方法`putVal`

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
               boolean evict) {  
    Node<K,V>[] tab; Node<K,V> p; int n, i;  
    if ((tab = table) == null || (n = tab.length) == 0)  
        // 数组未初始化或未有元素，调用resize，初始化数组
        n = (tab = resize()).length;  
    // 哈希求模优化，求模转化为按位与
    if ((p = tab[i = (n - 1) & hash]) == null) 
        // 链表第一个元素，直接放进数据即可 
        tab[i] = newNode(hash, key, value, null);  
    else {  
        Node<K,V> e; K k;  
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            // 链表已有元素，第一个元素是相同的key，取出原来的Node
            e = p;  
        else if (p instanceof TreeNode)  
            // 链表已转为红黑树，将值存放到红黑树中，若有旧值则返回
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        else { 
            // 链表已有元素，第一个元素不是相同的key，从链表的第二个元素开始遍历
            for (int binCount = 0; ; ++binCount) {  
                if ((e = p.next) == null) {  
                    // 在链表中找不到相同key的元素，链表尾加上新的元素
                    p.next = newNode(hash, key, value, null);  
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        // 加入新元素后，链表元素个数超过TREEIFY_THRESHOLD限制值，转化为红黑树
                        treeifyBin(tab, hash);  
                    break;                }  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    // 在链表中找到相同key的元素，此时e指向旧值
                    break; 
                // 遍历链表移动指针     
                p = e;  
            }  
        }  
        if (e != null) { // existing mapping for key  
            // 存在旧值，e不为空
            V oldValue = e.value;  
            if (!onlyIfAbsent || oldValue == null)  
                // 替换新值
                e.value = value;  
            afterNodeAccess(e);  
            return oldValue;  
        }  
    }  
    ++modCount;  
    if (++size > threshold)  
        // 放入元素后，元素总个数大于扩容阈值，调用resize
        resize();  
    afterNodeInsertion(evict);  
    return null;
}
```

## 查找元素

查找`HashMap`中的元素的核心方法`getNode`

```java
final Node<K,V> getNode(int hash, Object key) {  
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (first = tab[(n - 1) & hash]) != null) {  
        // 通过哈希求模获取元素位置，该位置的链表不为空
        if (first.hash == hash && // always check first node  
            ((k = first.key) == key || (key != null && key.equals(k))))  
            // 要查找的元素的key与链表中的第一个元素的key相等，查找成功
            return first;  
        if ((e = first.next) != null) {  
            if (first instanceof TreeNode)  
                // 红黑树查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);  
            do {  
                // 遍历链表中的所有元素,找到匹配的元素则查找成功
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
    }  
    return null;  
}
```

## 哈希扰动

`HashMap`在计算key的哈希值时，先调用key的`hashCode`函数，为了防止效果不好的哈希函数冲突过多，`HashMap`会在此结果的基础上再做一次哈希扰动

```java
static final int hash(Object key) {  
    int h;
    // 将原始key的哈希值与自身逻辑右移16位的结果按位异或
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```