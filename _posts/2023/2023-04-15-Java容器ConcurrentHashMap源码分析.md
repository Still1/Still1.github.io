---
title: Java容器ConcurrentHashMap源码分析
tags: [Java, 源码分析, 概念原理, 多线程]
mathjax: true
---

本文源码基于JDK8
{:.info}

## 优化算法

以下在`HashMap`中使用的优化算法在`ConcurrentHashMap`中也同样使用了，详情请查看

* [哈希求模优化](https://blog.oliverclio.com/2019/09/10/Java%E5%AE%B9%E5%99%A8HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html#%E5%93%88%E5%B8%8C%E6%B1%82%E6%A8%A1%E4%BC%98%E5%8C%96)
* [扩容哈希求模优化](https://blog.oliverclio.com/2019/09/10/Java%E5%AE%B9%E5%99%A8HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html#%E6%89%A9%E5%AE%B9%E5%93%88%E5%B8%8C%E6%B1%82%E6%A8%A1%E4%BC%98%E5%8C%96)
* [哈希扰动](https://blog.oliverclio.com/2019/09/10/Java%E5%AE%B9%E5%99%A8HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html#%E5%93%88%E5%B8%8C%E6%89%B0%E5%8A%A8)

其中哈希扰动算法在`HashMap`的基础上再进行了一点优化，加入了哈希掩码

```java
static final int HASH_BITS = 0x7fffffff;
```

将原来的哈希值与哈希掩码按位与，相当于将原来的哈希值的最高位置0，可进一步降低哈希冲突的可能性

```java
static final int spread(int h) {  
    return (h ^ (h >>> 16)) & HASH_BITS;  
}
```

## 数据结构

`ConcurrentHashMap`的基础数据结构为链表数组，链表节点通过`Node`类型来表示

```java
transient volatile Node<K,V>[] table;
```

`Node`类型实现了`Entry`接口，可存放键值对，链表的一个节点也就是一个键值对

```java
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    // 并发操作，保证可见性
    volatile V val;  
    volatile Node<K,V> next;  
  
    Node(int hash, K key, V val, Node<K,V> next) {  
        this.hash = hash;  
        this.key = key;  
        this.val = val;  
        this.next = next;  
    }  
  
    public final K getKey()       { return key; }  
    public final V getValue()     { return val; }  
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }  
    public final String toString(){ return key + "=" + val; }  
    public final V setValue(V value) {  
        throw new UnsupportedOperationException();  
    }  
  
    public final boolean equals(Object o) {  
        Object k, v, u; Map.Entry<?,?> e;  
        return ((o instanceof Map.Entry) &&  
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&  
                (v = e.getValue()) != null &&  
                (k == key || k.equals(key)) &&  
                (v == (u = val) || v.equals(u)));  
    }  
  
    /**  
     * Virtualized support for map.get(); overridden in subclasses.
     */
     // 通过哈希值和key对象查找元素，此处实现的是在链表中遍历查找，子类会重写这个方法    
     Node<K,V> find(int h, Object k) {  
        Node<K,V> e = this;  
        if (k != null) {  
            do {  
                K ek;  
                if (e.hash == h &&  
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
        return null;  
    }  
}
```

默认情况下，`ConcurrentHashMap`加入一个新元素后，一个链表元素个数大于8，且容器大小大于等于64时，链表将转化为红黑树

```java
static final int TREEIFY_THRESHOLD = 8;
static final int MIN_TREEIFY_CAPACITY = 64;
```

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {  
    Node<K,V> b; int n, sc;  
    if (tab != null) {  
        // 容器大小小于转化限制时，不转化红黑树，而是先扩容
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)  
            tryPresize(n << 1);  
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {  
            synchronized (b) {  
                if (tabAt(tab, index) == b) {  
                    TreeNode<K,V> hd = null, tl = null;  
                    for (Node<K,V> e = b; e != null; e = e.next) {  
                        TreeNode<K,V> p =  
                            new TreeNode<K,V>(e.hash, e.key, e.val,  
                                              null, null);  
                        if ((p.prev = tl) == null)  
                            hd = p;  
                        else                            tl.next = p;  
                        tl = p;  
                    }  
                    setTabAt(tab, index, new TreeBin<K,V>(hd));  
                }  
            }  
        }  
    }  
}
```

默认情况下，红黑树节点个数小于等于6，红黑树转化为链表

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

数组中的一个元素，代表一个链表或一颗红黑树，我们也可以统称其为一个桶（bin)
{:.info}

## 初始化

`ConcurrentHashMap`的容器容量是指数组的大小，也可理解为桶的数量

`ConcurrentHashMap`默认情况下初始容量为16

```java
private static final int DEFAULT_CAPACITY = 16;
```

`ConcurrentHashMap`的容量大小必须满足$2^n\ \ (n \ge 0)$，最大容量为$2^{30}$，这是int范围内最大的符合条件的数字。int范围内最大数字为$2^{31}-1$，不符合条件

```java
private static final int MAXIMUM_CAPACITY = 1 << 30;
```

`sizeCtl`变量控制数组的初始化和扩容，`sizeCtl`的值有以下几种情况
* `sizeCtl`的值为-1,则数组正在初始化
* `sizeCtl`的值为负数，且不等于-1，则数组正在扩容，且低16位表示的值为正在扩容的线程数 + 1
* 数组未初始化时，`sizeCtl`的值为初始化数组的大小，若构造函数未指定，则为0
* 数组初始化后，`sizeCtl`的值为触发下一次扩容的元素个数

```java
private transient volatile int sizeCtl;
```

`ConcurrentHashMap`无参构造函数，以默认初始容量16，创建一个`ConcurrentHashMap`

```java
public ConcurrentHashMap() {  
}
```

`ConcurrentHashMap`完整参数构造函数

`ConcurrentHashMap`构造函数中的`initialCapacity`参数的含义与`HashMap`构造函数并不一致
{:.warning}

`ConcurrentHashMap`构造函数中的`initialCapacity`参数代表的是不经过扩容保证可以存放元素的数量
{:.warning}

`HashMap`构造函数中的`initialCapacity`参数代表的是容器容量（即桶数量）至少需要达到的数量
{:.warning}

`ConcurrentHashMap`构造函数中的`loadFactor`参数仅用于构造函数中的计算，而`HashMap`构造函数中的`loadFactor`参数会作为`HashMap`的一个属性保存下来，影响构造函数以及后续操作中的扩容阈值计算
{:.warning}

```java
public ConcurrentHashMap(int initialCapacity,  
                         float loadFactor, int concurrencyLevel) {  
    // 处理初始化参数非法值
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)  
        throw new IllegalArgumentException();  
    // concurrencyLevel是一个并发线程数的预测值，仅用于提示初始化容量
    // 保证可以存放元素的数量至少是concurrencyLevel
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins  
        initialCapacity = concurrencyLevel;   // as estimated threads  
    // 根据保证可以存放元素的数量和扩容因子，求出至少需要的容器容量
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);  
    // 容器容量需要保证是2的n次幂（n大于等于0）
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?  
        MAXIMUM_CAPACITY : tableSizeFor((int)size);  
    // 初始化的容器容量存放在sizeCtl变量中
    this.sizeCtl = cap;  
}
```

`tableSizeFor`方法的逻辑可参考[[Java容器HashMap源码分析#初始化]]

`ConcurrentHashMap`的数组初始化在需要放入元素的时候才进行，数组初始化通过`initTable`方法来完成

```java
private final Node<K,V>[] initTable() {  
    Node<K,V>[] tab; int sc;  
    while ((tab = table) == null || tab.length == 0) {  
        // 数组尚未初始化
        if ((sc = sizeCtl) < 0)  
            // sizeCtl变量小于0，表示其它线程正在初始化，本线程先让出当前时间片
            Thread.yield(); // lost initialization race; just spin  
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  
            // sizeCtl变量大于或等于0，且CAS成功修改值为-1，开始初始化
            try {  
                if ((tab = table) == null || tab.length == 0) {  
                    // 构造函数指定了数组初始化大小，则按此大小初始化，若未指定，则按默认大小
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;  
                    @SuppressWarnings("unchecked")  
                    // 初始化数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  
                    table = tab = nt;  
                    // 计算触发下次扩容的元素个数，没有loadFactor的参与，固定为0.75 * n
                    sc = n - (n >>> 2);  
                }  
            } finally {  
                // 若初始化成功，更新sizeCtl值为触发下次扩容的元素个数
                // 若初始化失败，则更新sizeCtl值为旧值，相当于单纯释放锁
                sizeCtl = sc;  
            }  
            break;  
        }  
    }  
    return tab;  
}
```

## 数组元素访问

带有volatile语义的数组元素读取

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {  
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);  
}
```

带有volatile语义的数组元素写入

```java
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {  
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);  
}
```

对数组元素的CAS操作，读写都带有volatile语义

```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,  
                                    Node<K,V> c, Node<K,V> v) {  
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);  
}
```

## 扩容戳

调用`resizeStamp`计算扩容戳，扩容戳可理解为此次扩容的唯一标识

```java
// 扩容戳占用的二进制位数，默认是16位，最少需要6位
private static final int RESIZE_STAMP_BITS = 16;
// 扩容戳存放在sizeCtl高位时需要逻辑右移的位数，默认为16位
private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
```

```java
static final int resizeStamp(int n) {  
    // 扩容前容量的前置0个数代表扩容前容量，放置在扩容戳的低位，并将扩容戳的最高位设置为1
    return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));  
}
```

扩容戳与扩容前的容量大小和扩容戳占用的二进制位数有关。由于`CurrentHashMap`的容量大小必须满足$2^n\ \ (n \ge 0)$，且容量大小为一个整型变量（确定是32个二进制位），因此扩容前容量大小的前置0个数可以唯一确定一个容量大小值。扩容戳中使用扩容前容量的前置0个数代表扩容前容量，放置在扩容戳的低位，并将扩容戳的最高位设置为1

一个计算扩容戳的简单例子，扩容前的容量大小为32，整型变量32二进制表示为

$$0000\ 0000\ 0000\ 0000\ 0000\ 0000\ 0010\ 0000$$

可以看出整型变量32的二进制数有26个前置0，假设扩容戳占用的二进制位数为16位，将26放在扩容戳的低位，并将扩容戳最高位设置为1，得出扩容戳

$$ 1000\ 0000\ 0001\ 1010$$

开始扩容前，需要将`sizeCtl`变量修改为表示扩容中状态的格式，高16位为扩容戳，低16位为正在扩容的线程数加1

```java
U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2);
```

修改后的`sizeCtl`值

$$ 1000\ 0000\ 0001\ 1010\ 0000\ 0000\ 0000\ 0010$$

扩容线程最大值

```java
private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
```

假如扩容戳占用16位，即`sizeCtl`中表示正在扩容的线程数也占16位，有扩容线程最大值为

$$ 1111\ 1111\ 1111\ 1111$$

## 扩容核心

```java
// 表示数组元素已被转移的特殊哈希值
static final int MOVED = -1;
```

扩容过程中，原数组已转移的元素，需要放置一个`forwardingNode`标识已转移状态

```java
static final class ForwardingNode<K,V> extends Node<K,V> {  
    final Node<K,V>[] nextTable;  
    ForwardingNode(Node<K,V>[] tab) {  
        // 节点特殊的哈希值表示元素已转移
        super(MOVED, null, null, null);  
        this.nextTable = tab;  
    }  

    // 正在扩容时，元素已从原数组转移，通过此方法查找元素
    Node<K,V> find(int h, Object k) {  
        // loop to avoid arbitrarily deep recursion on forwarding nodes  
        outer: for (Node<K,V>[] tab = nextTable;;) {  
            Node<K,V> e; int n;  
            if (k == null || tab == null || (n = tab.length) == 0 ||  
                (e = tabAt(tab, (n - 1) & h)) == null)  
                return null;  
            for (;;) {  
                int eh; K ek;  
                if ((eh = e.hash) == h &&  
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))  
                    return e;  
                if (eh < 0) {  
                    if (e instanceof ForwardingNode) {  
                        tab = ((ForwardingNode<K,V>)e).nextTable;  
                        continue outer;  
                    }  
                    else  
                        return e.find(h, k);  
                }  
                if ((e = e.next) == null)  
                    return null;  
            }  
        }  
    }  
}
```

```java
// 扩容时单次分配的最少工作量，工作量指转移到新扩容数组的元素数量
private static final int MIN_TRANSFER_STRIDE = 16;
```

扩容核心方法`transfer`，所有扩容最终都是通过此方法实现，每次调用`transfer`都会把容量扩大到原来2倍

整个扩容过程为，由其中某一个线程启动扩容，新建一个原数组长度2倍的新数组，然后把原数组中的元素逐一转移到新数组。其它线程有可能加入到这个扩容过程中，每一条线程（包括启动扩容的线程）在转移数组中的元素之前，都要先确认属于当前线程的本轮转移元素的范围，然后逐一转移该范围的元素到新数组，待完成该范围的工作量后，再重新确认新一轮转移元素的范围并继续转移，直到所有元素都转移到新数组，新数组替代原数组，扩容完成

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {  
    int n = tab.length, stride;  
    // 根据逻辑处理器数量计算单次分配的转移到新扩容数组的元素数量，不能小于限制值
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)  
        stride = MIN_TRANSFER_STRIDE; // subdivide range  
    if (nextTab == null) {            // initiating  
        // 新开始一个扩容过程
        try {  
            @SuppressWarnings("unchecked")  
            // 开始扩容，创建原数组2倍长度的新数组
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];  
            nextTab = nt;  
        } catch (Throwable ex) {      // try to cope with OOME  
            // 扩容异常，将触发下次扩容的元素数量设置到Integer.MAX_VALUE
            sizeCtl = Integer.MAX_VALUE;  
            return;        
        }  
        nextTable = nextTab;  
        // 原数组元素转移到新扩容数组工作量分配下标，从原数组尾部开始分配
        transferIndex = n;  
    }  
    int nextn = nextTab.length;  
    // 原数组中已转移的元素，放置一个forwardingNode标记已转移
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);  
    boolean advance = true;  
    boolean finishing = false; // to ensure sweep before committing nextTab  
    // 不断循环，直到扩容完毕
    for (int i = 0, bound = 0;;) {  
        Node<K,V> f; int fh;  
        while (advance) {  
            int nextIndex, nextBound;  
            if (--i >= bound || finishing)  
                // 上一次分配给当前线程的工作量未完成，--i移动要转移的数组元素下标，继续转移下一个数组元素
                // 或扩容已确认完成，准备收尾工作
                // 跳出工作分配循环
                advance = false;  
            else if ((nextIndex = transferIndex) <= 0) {  
                // 上一次分配给当前线程的工作量已完成，且扩容未确认完成
                // 且扩容工作量已分配完了
                // 标记当前线程扩容工作已完成，并跳出工作分配循环
                i = -1;  
                advance = false;  
            }  
            else if (U.compareAndSwapInt  
                     (this, TRANSFERINDEX, nextIndex,  
                      nextBound = (nextIndex > stride ?  
                                   nextIndex - stride : 0))) {  
                // 分配给当前线程的工作量已完成，且扩容未确认完成
                // 且扩容工作量未分配完
                // 且尝试分配工作量给当前线程成功
                // 重新计算转移的数组元素的下标与边界
                // 跳出工作分配循环
                bound = nextBound;  
                i = nextIndex - 1;  
                advance = false;  
            }  
        }  
        if (i < 0 || i >= n || i + n >= nextn) {  
            // 当前线程扩容工作已完成
            int sc;  
            if (finishing) {  
                // 扩容确认完成，扩容的新数组成为正式数组
                nextTable = null;  
                table = nextTab;  
                // 重新计算触发下次扩容的元素数量，为0.75 * n
                // 赋值给sizeCtl，取消扩容状态标识，完成扩容
                sizeCtl = (n << 1) - (n >>> 1);  
                return;            
            }  
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {  
                // 扩容未确认完成
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)  
                    // 当前线程不是正在扩容的最后一条线程，当前线程退出
                    return;  
                // 当前线程是正在扩容的最后一条线程
                // 确认扩容已完成，后面进入工作分配循环
                finishing = advance = true;  
                i = n; // recheck before commit  
            }  
        }  
        else if ((f = tabAt(tab, i)) == null)  
            // 当前线程扩容工作未完成
            // 且当前要转移的数组元素为空
            // CAS尝试将forwardingNode放进原数组的对应位置
            // 若CAS操作成功，则对该数组元素的转移工作完成，再次进入工作分配循环
            // 若CAS操作失败，则再次尝试同样的处理
            advance = casTabAt(tab, i, null, fwd);  
        else if ((fh = f.hash) == MOVED)  
            // 当前线程扩容工作未完成
            // 且当前要转移的数组元素已被替换为forwardingNode，已处理完成
            // 再次进入工作分配循环
            advance = true; // already processed  
        else {  
            // 当前线程扩容工作未完成
            // 且当前要转移的数组元素存在，尚未被处理
            synchronized (f) {  
                // 获取互斥锁，以链表的第一个元素作为锁
                // 持有锁后再次检查是否有其它线程处理该元素
                if (tabAt(tab, i) == f) {  
                    // 将原数组元素的链表拆分成两条链表ln和hn
                    Node<K,V> ln, hn;  
                    if (fh >= 0) {  
                        // 数组元素是链表节点
                        // 根据节点hash值和原数组长度按位与的结果计算当前链表节点属于ln还是hn
                        int runBit = fh & n;  
                        Node<K,V> lastRun = f;  
                        // 遍历原链表，将原链表尾部部分配到同一链表的部分截出来
                        for (Node<K,V> p = f.next; p != null; p = p.next) {  
                            int b = p.hash & n;  
                            if (b != runBit) {  
                                runBit = b;  
                                lastRun = p;  
                            }  
                        }
                        // 截出来的链表先放到ln或hn
                        if (runBit == 0) {  
                            ln = lastRun;  
                            hn = null;  
                        }  
                        else {  
                            hn = lastRun;  
                            ln = null;  
                        }  
                        // 再次遍历原链表，忽略上一步已处理的尾部部分
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {  
                            int ph = p.hash; K pk = p.key; V pv = p.val;  
                            // 计算链表节点属于ln还是hn,并复制一个链表节点，加入到新链表中
                            if ((ph & n) == 0)  
                                ln = new Node<K,V>(ph, pk, pv, ln);  
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);  
                        }  
                        // 将新链表放到扩容数组中
                        setTabAt(nextTab, i, ln);  
                        setTabAt(nextTab, i + n, hn);  
                        // 原数组该元素的转移工作已完成，替换成forwardingNode
                        setTabAt(tab, i, fwd);  
                        // 再次进入工作分配循环
                        advance = true;  
                    }  
                    else if (f instanceof TreeBin) {  
                        // 当前数组元素是红黑树节点
                        // 拆分成两颗红黑树lo和hi
                        TreeBin<K,V> t = (TreeBin<K,V>)f;  
                        TreeNode<K,V> lo = null, loTail = null;  
                        TreeNode<K,V> hi = null, hiTail = null;  
                        int lc = 0, hc = 0; 
                        // 遍历原红黑树 
                        for (Node<K,V> e = t.first; e != null; e = e.next) {  
                            int h = e.hash;  
                            // 复制红黑树节点
                            TreeNode<K,V> p = new TreeNode<K,V>  
                                (h, e.key, e.val, null, null);  
                            // 根据节点hash值和原数组长度按位与的结果计算当前红黑树节点属于lo还是hi
                            if ((h & n) == 0) {  
                                if ((p.prev = loTail) == null)  
                                    lo = p;  
                                else                                    
                                    loTail.next = p;  
                                loTail = p;  
                                ++lc;  
                            }  
                            else {  
                                if ((p.prev = hiTail) == null)  
                                    hi = p;  
                                else                                    
                                    hiTail.next = p;  
                                hiTail = p;  
                                ++hc;  
                            }  
                        }  
                        // 元素个数不足的红黑树需转化为链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :  
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;  
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :  
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;  
                        setTabAt(nextTab, i, ln);  
                        setTabAt(nextTab, i + n, hn);  
                        // 原数组该元素的转移工作已完成，替换成forwardingNode
                        setTabAt(tab, i, fwd);  
                        // 再次进入工作分配循环
                        advance = true;  
                    }  
                }  
            }  
        }  
    }  
}
```

其它线程发现正在扩容，调用`helpTransfer`协助扩容

```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {  
    Node<K,V>[] nextTab; int sc;  
    // 检查参数是否合法
    if (tab != null && (f instanceof ForwardingNode) &&  
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {  
        // 计算扩容戳
        int rs = resizeStamp(tab.length);  
        while (nextTab == nextTable && table == tab &&  
               (sc = sizeCtl) < 0) {  
            // 正在扩容
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
                sc == rs + MAX_RESIZERS || transferIndex <= 0)  
                // 扩容中的扩容戳与当前线程计算的不相等
                // 或扩容中的线程数为0，扩容已结束
                // 或扩容中的线程数达到最大值
                // 或扩容工作量已全部分配完了
                // 结束当前线程的扩容工作
                break;  
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {  
                // CAS修改sizeCtl成功，将扩容线程数加1，当前线程加入扩容
                transfer(tab, nextTab);  
                break;            
            }  
        }  
        return nextTab;  
    }  
    return table;  
}
```

尝试预扩容

```java
// 参数size，预扩容后需要保证可以存放的元素数量
private final void tryPresize(int size) {  
    // 根据需要保证存放的元素数量，计算至少需要的扩容后容量大小
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :  
        tableSizeFor(size + (size >>> 1) + 1);  
    int sc;  
    // 循环扩容直到满足条件
    while ((sc = sizeCtl) >= 0) {  
        // 没有其它线程正在初始化或扩容数组
        Node<K,V>[] tab = table; int n;  
        if (tab == null || (n = tab.length) == 0) {  
            // 数组未初始化
            // 若已有指定的初始化容量，则与前面计算的扩容后容量大小相比，取较大值作初始化
            n = (sc > c) ? sc : c;  
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  
                // CAS修改sizeCtl为-1成功，开始初始化
                try {  
                    // 检查是否有其它线程已初始化
                    if (table == tab) {  
                        // 没有其它线程初始化，开始初始化
                        @SuppressWarnings("unchecked")  
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  
                        table = nt;  
                        // 计算触发下次扩容的元素数量，为0.75 * n
                        sc = n - (n >>> 2);  
                    }  
                } finally {  
                    // 若初始化成功，更新sizeCtl值为触发下次扩容的元素个数
                    // 若初始化失败，则更新sizeCtl值为旧值，相当于单纯释放锁
                    sizeCtl = sc;  
                }  
            }  
        }  
        else if (c <= sc || n >= MAXIMUM_CAPACITY)  
            // 数组已存在
            // 计算的扩容后容量小于等于触发下次扩容的元素个数
            // 或当前容量已达最大值
            // 不需要扩容
            break;  
        else if (tab == table) {  
            // 数组已存在且未有其它线程在当前循环开始后已完成扩容
            // 计算扩容戳
            int rs = resizeStamp(n);  
            if (sc < 0) {  
                // 其它线程正在扩容
                Node<K,V>[] nt;  
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||  
                    transferIndex <= 0)  
                    // 扩容中的扩容戳与当前线程计算的不相等
                    // 或扩容中的线程数为0，扩容已结束
                    // 或扩容中的线程数达到最大值
                    // 或扩容数组尚未创建
                    // 或扩容工作量已全部分配完了
                    // 结束当前线程的扩容工作
                    break;  
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))  
                    // CAS修改sizeCtl成功，将扩容线程数加1，当前线程加入扩容
                    transfer(tab, nt);  
            }  
            else if (U.compareAndSwapInt(this, SIZECTL, sc,  
                                         (rs << RESIZE_STAMP_SHIFT) + 2))  
                // 目前没有其它线程正在扩容，且CAS修改sizeCtl表示正在扩容状态成功，开始扩容
                // 表示正在扩容状态的sizeCtl，高16位为扩容戳，低16位为正在扩容的线程数加1
                transfer(tab, null);  
        }  
    }  
}
```

源码中判断扩容中的线程数为0，扩容已结束的条件`sc == rs + 1`和判断扩容中的线程数达到最大值的条件`sc == rs + MAX_RESIZERS`是不对的，实际应为`sc == （rs <<< RESIZE_STAMP_SHIFT) + 1`和`sc == (rs <<< RESIZE_STAMP_SHIFT) + MAX_RESIZERS`
{:.warning}

JDK12修复了这个bug
{:.warning}

## 计数

`size`方法获取元素个数，`size`方法实现`Map`接口的方法，返回的是`int`类型的值，因此需要把返回值限制在`int`类型的范围内

```java
public int size() {  
    // 实际调用sumCount方法进行计数
    long n = sumCount();  
    return ((n < 0L) ? 0 :  
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :  
            (int)n);  
}
```

由于`ConcurrentHashMap`的元素个数可以大于`Integer.MAX_VALUE`，因此提供了另外一个`mappingCount`方法返回`long`类型元素个数

```java
public long mappingCount() {  
    // 实际调用sumCount方法进行计数
    long n = sumCount();  
    return (n < 0L) ? 0L : n; // ignore transient negative values  
}
```

`size`方法和`mappingCount`方法实际都是调用`sumCount`方法获取`ConcurrentHashMap`的元素个数

```java
final long sumCount() {  
    // 将CounterCell数组中的所有元素的value值和baseCount加起来，得到元素个数
    CounterCell[] as = counterCells; CounterCell a;  
    long sum = baseCount;  
    if (as != null) {  
        for (int i = 0; i < as.length; ++i) {  
            if ((a = as[i]) != null)  
                sum += a.value;  
        }  
    }  
    return sum;
}
```

`ConcurrentHashMap`计数部分引入`CounterCell`数组，目的是提高性能。假如只使用一个`baseCount`变量来记录元素个数，则在高并发环境下对`baseCount`变量操作的竞争会很大。而引入`CounterCell`数组后，目标是尽可能地实现各个线程各自操作一个`CounterCell`，避免竞争

```java
@sun.misc.Contended static final class CounterCell {  
    // 记录元素个数的变量
    volatile long value;  
    CounterCell(long x) { value = x; }  
}
```

`CounterCell`使用了`@sun.misc.Contended`注解，作用是提高性能
{:.info}

`CounterCell`对象被预期经常发生多线程并发改动，`CounterCell`对象在高速缓存中所对应的缓存行也会经常失效，导致与`CounterCell`对象在同一缓存行的其它内容很难命中缓存，即使这些内容总是不变的。这种情况被称为伪共享（false sharing）
{:.info}

由于`CounterCell`总是以数组的形式存在，与某一`CounterCell`对象在同一缓存行的很有可能是同一数组中的其它`CounterCell`对象，这导致缓存行失效的情况更严重
{:.info}

使用`@sun.misc.Contended`注解，标记`CounterCell`对象被预期经常发生多线程并发改动，可以使一个`CounterCell`对象独享一个缓存行，从而避免伪共享问题
{:.info}

调用`addCount`方法修改`ConcurrentHashMap`元素个数，并检查是否需要扩容

```java
// 参数x，表示增加的元素个数
// 参数check，与是否检查扩容有关
private final void addCount(long x, int check) {  
    CounterCell[] as; long b, s;  
    if ((as = counterCells) != null ||  
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {  
        // 已存在counterCells 或 不存在counterCells且CAS操作baseCount失败
        // 已存在counterCells，优先使用counterCells进行计数
        // CAS操作baseCount失败，表示有并发修改，需初始化counterCells
        CounterCell a; long v; int m;  
        boolean uncontended = true;  
        if (as == null || (m = as.length - 1) < 0 ||  
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||  
            !(uncontended =  
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {  
            // counterCells不存在，或counterCells长度为0，调用fullAddCount初始化counterCells
            // counterCells存在且长度大于0，使用线程的probe随机数充当类似线程哈希值的角色
            // 通过ThreadLocalRandom.getProbe() & m 计算当前线程匹配conterCells的元素位置
            // 假如该匹配位置为空，则调用fullAddCount初始化该位置的counterCell对象
            // 假如该匹配位置不为空，则尝试CAS修改该位置counterCell对象的value值
            // 若CAS操作失败，则调用fullAddCount作后续操作，且uncontended的值为false，表示操作counterCell对象有竞争
            fullAddCount(x, uncontended);  
            // 通过fullAddCount处理计数的，不检查是否需要扩容
            return;
        }  
        // 尝试CAS修改匹配位置counterCell对象的value值成功
        if (check <= 1)  
            // 增加元素的节点为链表的第一个节点或第二个节点，此处不检查是否需要扩容
            return;  
        // 计算一下元素总数，准备检查扩容
        s = sumCount();  
    }  
    // 不存在counterCells且CAS操作baseCount成功
    // 或存在counterCells，尝试CAS修改匹配位置counterCell对象的value值成功，且check > 1
    // check > 1意味着增加的元素在链表中的位置是第三个以及以后的节点，或者是一个红黑树节点
    if (check >= 0) {  
        // 增加了元素都需要检查是否需要扩容，移除元素的check为-1
        Node<K,V>[] tab, nt; int n, sc;  
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&  
               (n = tab.length) < MAXIMUM_CAPACITY) {  
            // 元素总数大于等于扩容阈值sizeCtl，且数组table不为空，且数组table的长度少于最大容量限制，则触发扩容
            // 计算扩容戳
            int rs = resizeStamp(n);  
            if (sc < 0) {  
                // 目前其它线程正在扩容中
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||  
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||  
                    transferIndex <= 0)  
                    // 扩容中的扩容戳与当前线程计算的不相等
                    // 或扩容中的线程数为0，扩容已结束
                    // 或扩容中的线程数达到最大值
                    // 或扩容数组尚未创建
                    // 或扩容工作量已全部分配完了
                    // 结束当前线程的扩容工作
                    break;  
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))  
                    // CAS修改sizeCtl成功，将扩容线程数加1，当前线程加入扩容
                    transfer(tab, nt);  
            }  
            else if (U.compareAndSwapInt(this, SIZECTL, sc,  
                                         (rs << RESIZE_STAMP_SHIFT) + 2))  
                // 目前没有其它线程正在扩容，且CAS修改sizeCtl表示正在扩容状态成功，开始扩容
                // 表示正在扩容状态的sizeCtl，高16位为扩容戳，低16位为正在扩容的线程数加1
                transfer(tab, null);
            // 扩容完毕后，重新计算元素总数，再次进入循环，检查是否需要再次扩容  
            s = sumCount();  
        }  
    }  
}
```

`addCount`中的一些复杂计数操作将调用`fullAddCount`进行处理

```java
// 当前机器的可用逻辑处理器数，counterCells数组长度大于等于可用逻辑处理器数后，不可再扩容
static final int NCPU = Runtime.getRuntime().availableProcessors();
```

```java
// 参数x，表示增加的元素个数
// 参数wasUncontended，表示addCount方法中对counterCell对象的修改是否未发生过线程间的争抢
// 若在addCount中发生过线程间争抢，则wasUncontended的值为false
// fullAddCount中的处理是，若wasUncontended的值为false，则重新生成线程probe随机数，然后将wasUncontended置true
private final void fullAddCount(long x, boolean wasUncontended) {  
    int h;  
    if ((h = ThreadLocalRandom.getProbe()) == 0) {  
        // 线程的probe随机数尚未初始化，先初始化probe
        ThreadLocalRandom.localInit();      // force initialization  
        h = ThreadLocalRandom.getProbe();  
        // 新生成了线程的probe随机数，与之前addCount方法中使用的默认probe值0不相同
        wasUncontended = true;  
    }  
    // 需要扩容counterCells的标识位
    boolean collide = false;                // True if last slot nonempty  
    // 不断循环，直到计数处理完毕
    for (;;) {  
        CounterCell[] as; CounterCell a; int n; long v;  
        if ((as = counterCells) != null && (n = as.length) > 0) {  
            // counterCells已存在且长度大于0
            if ((a = as[(n - 1) & h]) == null) {  
                // 使用当前线程probe随机数，匹配到的counterCells位置上的counterCell对象未初始化
                if (cellsBusy == 0) {            // Try to attach new Cell  
                    // 未有其它线程正在修改counterCells，互斥锁cellsBusy未被获取
                    // 创建一个counterCell对象，并把增加的元素个数x放进counterCell对象中
                    CounterCell r = new CounterCell(x); // Optimistic create  
                    // 再次检查互斥锁cellsBusy是否被其它线程获取，且CAS尝试获取互斥锁
                    if (cellsBusy == 0 &&  
                        U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {  
                        // 获取互斥锁成功，对conterCells进行修改
                        // 对应位置的counterCell对象初始化成功的标识
                        boolean created = false;  
                        try {               // Recheck under lock  
                            CounterCell[] rs; int m, j;  
                            // 持有互斥锁后重新检查，确保未被其它线程的操作影响
                            // 重新检查counterCells是否已存在且长度大于0
                            // 重新检查对应位置的counterCell对象是否未初始化
                            if ((rs = counterCells) != null &&  
                                (m = rs.length) > 0 &&  
                                rs[j = (m - 1) & h] == null) {  
                                // 重新检查确认未被其它线程的操作影响
                                // 将新创建的counterCell对象放入counterCells的对应位置
                                rs[j] = r;  
                                // 标识初始化counterCell对象成功
                                created = true;  
                            }  
                        } finally {  
                            // 无论初始化counterCell对象的操作是否成功，释放互斥锁
                            cellsBusy = 0;  
                        }  
                        if (created)  
                            // 初始化counterCell对象成功，本次计数处理完成，跳出循环
                            break;  
                        // 初始化counterCell对象失败，其它线程在当前线程持有锁前已初始化，再次进入循环
                        continue;           // Slot is now non-empty  
                    }  
                }
                // 获取counterCells互斥锁失败，counterCells扩容标识位collide置为false
                // 后面将重新生成一个线程的probe随机数，并再次进入循环
                collide = false;  
            }  
            else if (!wasUncontended)       // CAS already known to fail  
                // 当前线程probe随机数匹配到的counterCells位置上的counterCell对象已存在
                // 且wasUncontended的值为false，表示在addCount方法中曾经出现CAS操作失败，此匹配位置有其它线程争抢
                // 将wasUncontended置为true，后面将重新生成一个线程的probe随机数，并再次进入循环
                wasUncontended = true;      // Continue after rehash  
            else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))  
                // 当前线程probe随机数匹配到的counterCells位置上的counterCell对象已存在
                // 且wasUncontended的值为true
                // 尝试CAS修改匹配位置counterCell对象的value值成功，本次计数处理完成，跳出循环
                break;  
            else if (counterCells != as || n >= NCPU)  
                // 当前线程probe随机数匹配到的counterCells位置上的counterCell对象已存在
                // 且wasUncontended的值为true
                // 且尝试CAS修改匹配位置counterCell对象的value值失败
                // 且counterCells已被其它线程扩容成功或counterCells数组长度已达最大值，无法扩容
                // 将counterCells扩容标识位collide置为false，后面将重新生成一个线程的probe随机数，并再次进入循环
                collide = false;            // At max size or stale  
            else if (!collide)  
                // 当前线程probe随机数匹配到的counterCells位置上的counterCell对象已存在
                // 且wasUncontended的值为true
                // 且尝试CAS修改匹配位置counterCell对象的value值失败
                // 且counterCells未被其它线程成功扩容且counterCells数组长度未达最大值，可以扩容
                // 且counterCells扩容标识位collide为false
                // 将counterCells扩容标识位置为true，后面将重新生成一个线程的probe随机数，并再次进入循环
                collide = true;  
            else if (cellsBusy == 0 &&  
                     U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                // 当前线程probe随机数匹配到的counterCells位置上的counterCell对象已存在
                // 且wasUncontended的值为true
                // 且尝试CAS修改匹配位置counterCell对象的value值失败
                // 且counterCells未被其它线程成功扩容且counterCells数组长度未达最大值，可以扩容
                // 且counterCells扩容标识位collide为true
                // 且获取互斥锁cellsBusy成功
                try {  
                    // 处理counterCells扩容
                    // 持有锁后再次检查是否有其它线程已成功扩容
                    if (counterCells == as) { // Expand table unless stale  
                        // 未有其他线程已成功扩容
                        // 将counterCell元素复制到一个原来长度2倍的新counterCells数组
                        CounterCell[] rs = new CounterCell[n << 1];  
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];  
                        counterCells = rs;
                    }
                } finally {  
                    // 无论扩容是否成功，释放互斥锁
                    cellsBusy = 0;  
                }  
                // counterCells已由本线程或其它线程成功扩容，将counterCells扩容标识位置为false
                collide = false;  
                // 再次进入循环
                continue;                   // Retry with expanded table  
            }  
            // 重新生成一个线程的probe随机数，切换一个counterCells匹配位置，再次进入循环
            h = ThreadLocalRandom.advanceProbe(h);  
        }  
        else if (cellsBusy == 0 && counterCells == as &&  
                 U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {  
            // counterCells数组未初始化
            // 且counterCells数组未被其它线程初始化
            // 且获取互斥锁cellsBusy成功
            
            // counterCells数组初始化成功标识
            boolean init = false;  
            try {                           // Initialize table  
                // 持有锁后再次检查是否有其它线程已成功初始化
                if (counterCells == as) {  
                    // 未被其它线程成功初始化
                    // 初始化一个长度为2的counterCells数组
                    CounterCell[] rs = new CounterCell[2];  
                    // 在当前线程probe随机数匹配到的counterCells位置上初始化counterCell对象
                    // 并把增加的元素个数x放进counterCell对象中
                    rs[h & 1] = new CounterCell(x);  
                    counterCells = rs;  
                    // // counterCells数组初始化成功
                    init = true;  
                }  
            } finally {  
                // 无论初始化是否成功，释放互斥锁
                cellsBusy = 0;  
            }  
            if (init)  
                // 初始化counterCells数组成功，本次计数处理完成，跳出循环
                break;  
        }  
        else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))  
            // counterCells数组未初始化
            // 且获取互斥锁cellsBusy失败 或 再次检查时counterCells数组已被其它线程初始化
            // 且CAS修改baseCount变量成功
            // 本次计数处理完成，跳出循环
            break;                          // Fall back on using base  
    }  
}
```

## 放入元素

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {  
    // 不允许key为空或value为空
    if (key == null || value == null) throw new NullPointerException();  
    // 计算哈希值
    int hash = spread(key.hashCode());  
    int binCount = 0;  
    // 不断循环，直到处理完毕  
    for (Node<K,V>[] tab = table;;) {  
        Node<K,V> f; int n, i, fh;  
        if (tab == null || (n = tab.length) == 0)  
            // 数组未初始化
            tab = initTable();  
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  
            // 数组已初始化，但匹配位置的元素为空，CAS加入链表的第一个节点
            if (casTabAt(tab, i, null,  
                         new Node<K,V>(hash, key, value, null)))  
                // 放入元素成功，退出加入元素处理循环
                break;                   // no lock when adding to empty bin  
        }  
        else if ((fh = f.hash) == MOVED)  
            // 正在扩容，对应位置的数组元素已转移，加入协助扩容
            tab = helpTransfer(tab, f);  
        else {  
            V oldVal = null;
            // 获取锁，链表第一个节点作为锁  
            synchronized (f) { 
                // 可能存在并发修改，持有锁后先检查链表第一个节点是否已被修改，未被修改才继续操作 
                if (tabAt(tab, i) == f) {  
                    if (fh >= 0) {  
                        // 数组第一个节点是链表节点，hash的值大于0
                        // 通过binCount记录遍历到链表的第几个节点
                        binCount = 1;  
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {  
                            K ek;  
                            // 找到key相同的元素 
                            if (e.hash == hash &&  
                                ((ek = e.key) == key ||  
                                 (ek != null && key.equals(ek)))) {  
                                // 记录旧值
                                oldVal = e.val;  
                                if (!onlyIfAbsent)  
                                    // 不是onlyIfAbsent，设置新值
                                    e.val = value;  
                                // 退出链表遍历循环
                                break;                            
                            }  
                            Node<K,V> pred = e;  
                            if ((e = e.next) == null) {  
                                // 遍历完链表找不到key相同的元素，链表尾部加入一个新的元素节点
                                pred.next = new Node<K,V>(hash, key,  
                                                          value, null);  
                                // 退出链表遍历循环
                                break;                            
                            }  
                        }  
                    }  
                    else if (f instanceof TreeBin) {  
                        // 已转化为红黑树
                        Node<K,V> p;
                        // 红黑树binCount固定为2  
                        binCount = 2;  
                        // 调用红黑树putTreeVal方法放入元素
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,  
                                                       value)) != null) {  
                            // 存在旧值，记录旧值
                            oldVal = p.val;  
                            if (!onlyIfAbsent)  
                                // 不是onlyIfAbsent，设置新值
                                p.val = value;  
                        }  
                    }  
                }  
            }  
            if (binCount != 0) {  
                if (binCount >= TREEIFY_THRESHOLD)  
                    // 链表元素数量达到阈值，转化为红黑树
                    treeifyBin(tab, i);  
                if (oldVal != null)  
                    // 存在旧值，则返回旧值，没有增加元素，不会调用addCount
                    return oldVal;  
                // 退出加入元素处理循环
                break;            
            }  
        }  
    }  
    // 元素数量增加1，没有元素增加的不会调用到这里
    // 增加的元素在链表中，binCount的值为链表的下标
    // 增加的元素在红黑树中，binCount的值为2
    addCount(1L, binCount);  
    return null;
}
```

## 查找元素

```java
public V get(Object key) {  
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;  
    // 计算哈希值
    int h = spread(key.hashCode());  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (e = tabAt(tab, (n - 1) & h)) != null) {  
        // 根据hash值匹配的数组位置不为空
        if ((eh = e.hash) == h) {  
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))  
                // 链表的第一个节点与查找的key相等，返回值
                return e.val;  
        }  
        else if (eh < 0)  
            // 数组对应位置元素的hash值小于0，代表已转化为红黑树，或者正在扩容，元素已转移到新扩容数组中
            // 通过node对象的find方法查找
            return (p = e.find(h, key)) != null ? p.val : null;  
        // 第一个节点不是要查找的元素，且不是红黑树，且不是正在扩容
        // 遍历链表剩余的元素
        while ((e = e.next) != null) {  
            if (e.hash == h &&  
                ((ek = e.key) == key || (ek != null && key.equals(ek))))  
                // 在链表中找到与查找的key相等的元素，返回值
                return e.val;  
        }  
    }  
    // 查找失败
    return null;  
}
```