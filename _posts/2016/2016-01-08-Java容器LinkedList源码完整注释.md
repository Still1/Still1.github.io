---
title: Java容器LinkedList源码完整注释
tags: [Java, 源码分析, 概念原理]
---

本文源码基于JDK8
{:.info}

## LinkedList类继承关系

![image-20220708125246311](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202207081252056.png)

## Iterator接口

```java
public interface Iterator<E> {
    // 检测是否还有下一个元素
    boolean hasNext();

    // 返回下一个元素
    E next();

    // 将上一次next方法返回的元素删除
    default void remove() {
        // 抛出异常
        throw new UnsupportedOperationException("remove");
    }

    // 对剩余的所有元素调用Consumer
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

## Iterable接口

```java
public interface Iterable<T> {
    
    // 返回一个用于遍历容器的Iterator
    Iterator<T> iterator();

    // 直接遍历容器，接口默认实现，对每个元素调用action
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    // 返回一个用于并行遍历容器的Spliterator
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

## Collection接口

```java
public interface Collection<E> extends Iterable<E> {
   
    // 查询操作
    
    // 返回容器元素数量
    int size();

    // 判断容器是否为空
    boolean isEmpty();

    // 判断容器是否包含某元素
    boolean contains(Object o);

    // 重复定义Iterable接口方法
    // 返回一个用于遍历容器的Iterator
    Iterator<E> iterator();

    // 容器转换为数组
    Object[] toArray();

    <T> T[] toArray(T[] a);

    // 修改操作

    // 增加一个元素
    boolean add(E e);

    // 删除一个元素
    boolean remove(Object o);


    // 批量操作

    // 判断是否包含另一个容器的所有元素
    boolean containsAll(Collection<?> c);

    // 加入另一个容器的所有元素
    boolean addAll(Collection<? extends E> c);

    // 删除另一个容器存在的所有元素
    boolean removeAll(Collection<?> c);

    // 删除符合条件的元素
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    // 保留另一个容器存在的所有元素
    boolean retainAll(Collection<?> c);

    // 清除容器所有元素
    void clear();


    // 比较和哈希

    // 判断两个容器是否相等
    boolean equals(Object o);

    // 计算哈希值
    int hashCode();

    // 返回一个用于并行遍历容器的Spliterator，重写了Iterable接口的默认实现
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    // 容器流处理
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    // 容器并行流处理
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```

## AbstractCollection抽象类

```java
public abstract class AbstractCollection<E> implements Collection<E> {
    
    // 构造方法，仅供子类调用
    protected AbstractCollection() {
    }
    
    // 重复定义Collection接口的iterator方法
    public abstract Iterator<E> iterator();

    // 重复定义Collection接口的size方法
    public abstract int size();

    public boolean isEmpty() {
        // 容器元素个数为0，判断为空，否则不为空
        return size() == 0;
    }
    
    public boolean contains(Object o) {
        // 使用iterator遍历所有元素
        Iterator<E> it = iterator();
        if (o==null) {
            // null与null比较
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            // 非null则使用equals方法判断是否相等
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    public Object[] toArray() {
        // 使用size方法估计需要的数组长度。并发修改的情况下，可能这并非最终的数组长度
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext())
                // 比原估计长度小
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        // 比原估计长度大，需要再调用finishToArray方法
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // 使用size方法估计需要的数组长度。并发修改的情况下，可能这并非最终的数组长度
        int size = size();
        // 传入参数提供的数组足够长度足够，则直接使用。否则根据提供的数组元素类型，按size方法的长度生成新的数组
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { 
                // 比原估计长度小
                if (a == r) {
                    // 使用的是传入参数的数组，后面补null
                    r[i] = null;
                } else if (a.length < i) {
                    // 假如传入参数提供的数组现在长度还是不够，则复制一个当前长度的数组
                    return Arrays.copyOf(r, i);
                } else {
                    // 假如传入参数提供的数组现在长度足够，使用回这个数组，把元素复制过去
                    System.arraycopy(r, 0, a, 0, i);
                    // 后面补null
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            // 强制转型
            r[i] = (T)it.next();
        }
        // 比原估计长度大，需要再调用finishToArray方法
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    // 最大数组大小。Java数组length属性类型是int
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    
    @SuppressWarnings("unchecked")
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        // 将iterator未复制完的元素继续复制到数组中
        // 数组r已满，下一个要处理的数组index为r.length
        int i = r.length;
        while (it.hasNext()) {
            // 设置cap值，用于检测当前index是否已超出数组边界
            int cap = r.length;
            if (i == cap) {
                // 当前处理的index超出数组边界，则扩容
                int newCap = cap + (cap >> 1) + 1;
                // 扩容后长度大于数组最大长度的处理，调用hugeCapacity方法
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();
        }
        // 复制完后如数组有未使用的空间，将其缩减
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }

    private static int hugeCapacity(int minCapacity) {
        // 最小扩容，只把长度加1
        // 长度为负数则表示长度已大于最大int，抛出错误
        if (minCapacity < 0)
            throw new OutOfMemoryError
                ("Required array size too large");
        // 长度比数组最大长度大，则扩容到最大int，否则扩容到数组最大长度
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    // 修改操作

    public boolean add(E e) {
        // 抛出异常，不支持此操作
        throw new UnsupportedOperationException();
    }

    public boolean remove(Object o) {
        // 使用iterator方法返回的iterator遍历容器
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    // 通过iterator的remove方法删除相应的元素
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    // 通过iterator的remove方法删除相应的元素
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }


    // 批处理操作

    public boolean containsAll(Collection<?> c) {
        // 对传入参数的容器各元素调用contains方法
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        // 将传入参数的容器元素逐一加入
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                // 通过iterator的remove方法删除相应的元素
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            // 与removeAll方法相反
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }


    //  字符串转换

    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            // 防止无限递归
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }
}
```

## ListIterator接口

```java
public interface ListIterator<E> extends Iterator<E> {

    // 检测是否有下一个元素
    boolean hasNext();

    // 返回下一个元素
    E next();

    // 检测是否有上一个元素
    boolean hasPrevious();

    // 返回上一个元素
    E previous();

    // 返回下一个元素的index
    int nextIndex();

    // 返回上一个元素的index
    int previousIndex();

    // 删除上一次调用next或previous方法返回的元素，且调用remove之前不能有调用过add
    void remove();

    // 替换上一次调用next或previous方法返回的元素，且调用set之前不能有调用add或remove
    void set(E e);

    // 在当前游标前面增加元素
    void add(E e);
}
```

## List接口

```java
public interface List<E> extends Collection<E> {

    // 重复定义Collection接口方法
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();
    
    // 重载Collection接口的addAll方法
    // 将c容器内的所有元素从List的index位置开始插入
    boolean addAll(int index, Collection<? extends E> c);

    // 对List每个元素进行替换操作
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        // 遍历容器所有元素
        while (li.hasNext()) {
            // 对每个元素施加操作，并将操作后的对象替换到List的原位置
            li.set(operator.apply(li.next()));
        }
    }

    // 排序
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        // 转数组并依据Comparator排序
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        // 将数组内的元素按顺序放回List
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    // 返回相应位置的元素
    E get(int index);

    // 替换相应位置的元素
    E set(int index, E element);

    // 在相应位置插入元素
    void add(int index, E element);

    // 删除相应位置的元素
    E remove(int index);

    // 返回第一次出现的索引值
    int indexOf(Object o);

    // 返回最后一次出现的索引值
    int lastIndexOf(Object o);

    // 返回ListIterator
    ListIterator<E> listIterator();

    // 返回ListIterator，开始位置为index
    ListIterator<E> listIterator(int index);

    // 返回子List，index范围包前不包后。子List依赖原List，数据改变会相互影响
    List<E> subList(int fromIndex, int toIndex);

    // 返回一个用于并行遍历容器的Spliterator，重写了Collection接口的默认实现
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}
```

## AbstractList抽象类

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    
    // 构造方法，仅供子类调用
    protected AbstractList() {
    }

    public boolean add(E e) {
        // 调用重载方法，新增元素放在尾部
        add(size(), e);
        return true;
    }

    // 重复定义List接口的get方法
    abstract public E get(int index);

    
    public E set(int index, E element) {
        // 抛出异常，不支持此操作
        throw new UnsupportedOperationException();
    }

    public void add(int index, E element) {
        // 抛出异常，不支持此操作
        throw new UnsupportedOperationException();
    }

    public E remove(int index) {
        // 抛出异常，不支持此操作
        throw new UnsupportedOperationException();
    }

    public int indexOf(Object o) {
        // 使用ListIterator遍历容器，找到元素则返回index
        ListIterator<E> it = listIterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        return -1;
    }

    public int lastIndexOf(Object o) {
        // 使用ListIterator反向遍历容器，找到元素则返回index
        ListIterator<E> it = listIterator(size());
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
    }

    // 重写了AbstractCollection的实现
    public void clear() {
        removeRange(0, size());
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            // 从index位置逐一添加元素
            add(index++, e);
            modified = true;
        }
        return modified;
    }

    // 返回Iterator
    public Iterator<E> iterator() {
        return new Itr();
    }

    // 返回起始位置为0的ListIterator
    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    // 返回起始位置为index的ListIterator
    public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);

        return new ListItr(index);
    }

    // 实现Iterator接口的内部类
    private class Itr implements Iterator<E> {
        // 下一次调用next要返回的元素index
        int cursor = 0;

        // 最近一次调用next或previous返回的元素的index，调用remove后将重置为-1
        int lastRet = -1;

        // Iterator期望List的modCount值，假如与真实modCount值不相等，则认为有并发修改
        int expectedModCount = modCount;

        public boolean hasNext() {
            // cursor值不等于size()，则认为有下个元素
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                // 获取游标所指的元素
                E next = get(i);
                lastRet = i;
                // 游标向后移动
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                // 调用List的remove方法删除元素
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    // 如remove之前调用的是next，lastRet == cursor - 1，调整游标
                    // 如remove之前调用的是previous，此时lastRet == cursor，不调整游标
                    cursor--;
                lastRet = -1;
                // List的modCount应该已改变，需要刷新expectedModCount
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        // 检查并发修改
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    // 实现ListIterator接口的内部类，继承了Itr类
    private class ListItr extends Itr implements ListIterator<E> {
        // 构造方法指定游标位置
        ListItr(int index) {
            cursor = index;
        }

        public boolean hasPrevious() {
            // 游标不为0，则认为存在前一个元素
            return cursor != 0;
        }

        public E previous() {
            checkForComodification();
            try {
                int i = cursor - 1;
                // 获取游标的前一个元素
                E previous = get(i);
                // 游标向前移动
                lastRet = cursor = i;
                return previous;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor-1;
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                // 调用List的set方法替换元素
                AbstractList.this.set(lastRet, e);
                // List的modCount应该已改变，需要刷新expectedModCount
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                // 调用List的add方法增加元素
                AbstractList.this.add(i, e);
                lastRet = -1;
                // 相当于在游标前插入，不影响next的遍历
                cursor = i + 1;
                // List的modCount应该已改变，需要刷新expectedModCount
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    public List<E> subList(int fromIndex, int toIndex) {
        // 根据当前List是否实现RamdonAccess确定子List的类型
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
    }

    // 都是List，长度相等，相同位置的元素相等，判断为相等
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator<?> e2 = ((List<?>) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }

    public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
    }

    // 范围删除元素，index包前不包后
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        // 从fromIndex开始删除，删除toIndex-fromIndex个元素
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }

    // List结构性改变次数标记
    protected transient int modCount = 0;

    // 检查index值是否超出添加元素范围，size()值合法
    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
    }
}

// 子List，依赖原List实现功能，并作范围限制
class SubList<E> extends AbstractList<E> {
    private final AbstractList<E> l;
    private final int offset;
    private int size;

    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
        l = list;
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;
    }

    public E set(int index, E element) {
        rangeCheck(index);
        checkForComodification();
        return l.set(index+offset, element);
    }

    public E get(int index) {
        rangeCheck(index);
        checkForComodification();
        return l.get(index+offset);
    }

    public int size() {
        checkForComodification();
        return size;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        checkForComodification();
        l.removeRange(fromIndex+offset, toIndex+offset);
        this.modCount = l.modCount;
        size -= (toIndex-fromIndex);
    }

    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        checkForComodification();
        l.addAll(offset+index, c);
        this.modCount = l.modCount;
        size += cSize;
        return true;
    }

    public Iterator<E> iterator() {
        return listIterator();
    }

    public ListIterator<E> listIterator(final int index) {
        checkForComodification();
        rangeCheckForAdd(index);

        return new ListIterator<E>() {
            private final ListIterator<E> i = l.listIterator(index+offset);

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return i.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return i.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return i.nextIndex() - offset;
            }

            public int previousIndex() {
                return i.previousIndex() - offset;
            }

            public void remove() {
                i.remove();
                SubList.this.modCount = l.modCount;
                size--;
            }

            public void set(E e) {
                i.set(e);
            }

            public void add(E e) {
                i.add(e);
                SubList.this.modCount = l.modCount;
                size++;
            }
        };
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new SubList<>(this, fromIndex, toIndex);
    }

    private void rangeCheck(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkForComodification() {
        if (this.modCount != l.modCount)
            throw new ConcurrentModificationException();
    }
}

// 相比SubList，多实现了RandomAccess接口
class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    RandomAccessSubList(AbstractList<E> list, int fromIndex, int toIndex) {
        super(list, fromIndex, toIndex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        return new RandomAccessSubList<>(this, fromIndex, toIndex);
    }
}
```

## AbstractSequentialList抽象类

```java
public abstract class AbstractSequentialList<E> extends AbstractList<E> {
    // 构造方法仅供子类调用
    protected AbstractSequentialList() {
    }

    // 使用listIterator实现各方法
    
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public E set(int index, E element) {
        try {
            ListIterator<E> e = listIterator(index);
            E oldVal = e.next();
            e.set(element);
            return oldVal;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public void add(int index, E element) {
        try {
            listIterator(index).add(element);
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public E remove(int index) {
        try {
            ListIterator<E> e = listIterator(index);
            E outCast = e.next();
            e.remove();
            return outCast;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        try {
            boolean modified = false;
            ListIterator<E> e1 = listIterator(index);
            Iterator<? extends E> e2 = c.iterator();
            while (e2.hasNext()) {
                e1.add(e2.next());
                modified = true;
            }
            return modified;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public Iterator<E> iterator() {
        return listIterator();
    }

    // 重新定义List接口的listIterator方法。父类AbstractList类实现了此方法
    public abstract ListIterator<E> listIterator(int index);
}
```

## Queue接口

```java
public interface Queue<E> extends Collection<E> {
    // 插入一个元素到队尾，无法插入则抛异常
    boolean add(E e);

    // 插入一个元素到队尾，无法插入则返回false
    boolean offer(E e);

    // 访问并移除队头的元素，队列为空无法移除则抛异常
    E remove();

    // 访问并移除队头的元素，队列为空无法移除则返回null
    E poll();

    // 访问头元素，队列为空则抛异常
    E element();

    // 访问头元素，队列为空则返回null
    E peek();
}
```

## Deque接口

```java
public interface Deque<E> extends Queue<E> {
    // 插入一个元素到双端队列的队头，无法插入则抛异常
    void addFirst(E e);

    // 插入一个元素到双端队列的队尾，无法插入则抛异常
    void addLast(E e);

    // 插入一个元素到双端队列的队头，无法插入则返回false
    boolean offerFirst(E e);

    // 插入一个元素到双端队列的队尾，无法插入则返回false
    boolean offerLast(E e);

    // 访问并移除队头的元素，队列为空无法移除则抛异常
    E removeFirst();

    // 访问并移除队尾的元素，队列为空无法移除则抛异常
    E removeLast();

    // 访问并移除队头的元素，队列为空无法移除则返回null
    E pollFirst();

    // 访问并移除队尾的元素，队列为空无法移除则返回null
    E pollLast();

    // 访问头元素，队列为空则抛异常
    E getFirst();

    // 访问尾元素，队列为空则抛异常
    E getLast();

    // 访问头元素，队列为空则返回null
    E peekFirst();

    // 访问尾元素，队列为空则返回null
    E peekLast();

    // 移除第一次出现的元素
    boolean removeFirstOccurrence(Object o);

    // 移除最后一次出现的元素
    boolean removeLastOccurrence(Object o);

    // 重复定义Queue接口的方法
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();

    // 将元素压入双端队列代表的栈，也就是放到队头。无法压入则抛异常
    void push(E e);

    // 将元素弹出双端队列代表的栈，也就是访问队头元素并移除。无法移除则抛异常
    E pop();


    // 重复定义Coolection接口的方法
    boolean remove(Object o);
    boolean contains(Object o);
    public int size();

    // 重复定义Iterable接口的方法
    Iterator<E> iterator();

    // 返回一个逆序的Iterator
    Iterator<E> descendingIterator();

}
```

## LinkedList

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    // LinkedList的长度
    transient int size = 0;

    // 指向链表的第一个元素的指针
    transient Node<E> first;

    // 指向链表最后一个元素的指针
    transient Node<E> last;

    // 构造一个空的LinkedList
    public LinkedList() {
    }

    // 构造一个包含c所有元素的LinkedList
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    // 将元素e连接到链表头
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

    // 将元素e连接到链表尾
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

    // 在节点succ前连接元素e
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

    // 解开链表头第一个节点
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

    // 解开链表尾最后一个节点
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    // 解开链表中的节点x
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }

    // 对底层链表操作实现Deque接口的方法
    
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }

    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    public void addFirst(E e) {
        linkFirst(e);
    }

    public void addLast(E e) {
        linkLast(e);
    }

    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    public int size() {
        return size;
    }

    // 对底层链表操作实现Collection接口的方法

    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }

    public void clear() {
        // 将链表节点之间的关联全部取消，帮助GC
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }

    // 基于node方法来定位实现位置相关的List接口方法
    
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }

    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }

    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

    // 检查边界
    
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    // 链表定位的关键方法
    Node<E> node(int index) {
        // assert isElementIndex(index);

        // 判断index位于链表前半部分还是后半部分，然后选择遍历入口
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

    // 从链表头开始遍历找元素
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }

    // 从链表尾开始遍历找元素
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }

    // 实现Queue接口方法
    
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    public E element() {
        return getFirst();
    }

    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    public E remove() {
        return removeFirst();
    }

    public boolean offer(E e) {
        return add(e);
    }

    // 对底层链表操作实现Deque接口的方法
    
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }

    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

    public void push(E e) {
        addFirst(e);
    }

    public E pop() {
        return removeFirst();
    }

    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    // 链表节点静态内部类
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    private class DescendingIterator implements Iterator<E> {
        // ListItr反向遍历
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }

    @SuppressWarnings("unchecked")
    private LinkedList<E> superClone() {
        try {
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    // 浅拷贝
    public Object clone() {
        LinkedList<E> clone = superClone();

        // Put clone into "virgin" state
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // Initialize clone with our elements
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }

    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }

    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(
                                a.getClass().getComponentType(), size);
        int i = 0;
        Object[] result = a;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;

        if (a.length > size)
            // 转化的数组还有空位，则在最后一个元素的后面赋值null
            // 在null不是合法值的数组内，可以帮助判断是否已遍历完该数组
            a[size] = null;

        return a;
    }

    private static final long serialVersionUID = 876323262645176354L;

    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out size
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.item);
    }

    @SuppressWarnings("unchecked")
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read in size
        int size = s.readInt();

        // Read in all elements in the proper order.
        for (int i = 0; i < size; i++)
            linkLast((E)s.readObject());
    }

    @Override
    public Spliterator<E> spliterator() {
        return new LLSpliterator<E>(this, -1, 0);
    }

    // 跳过Spliterator的分析
    /** A customized variant of Spliterators.IteratorSpliterator */
    static final class LLSpliterator<E> implements Spliterator<E> {
        static final int BATCH_UNIT = 1 << 10;  // batch array size increment
        static final int MAX_BATCH = 1 << 25;  // max batch array size;
        final LinkedList<E> list; // null OK unless traversed
        Node<E> current;      // current node; null until initialized
        int est;              // size estimate; -1 until first needed
        int expectedModCount; // initialized when est set
        int batch;            // batch size for splits

        LLSpliterator(LinkedList<E> list, int est, int expectedModCount) {
            this.list = list;
            this.est = est;
            this.expectedModCount = expectedModCount;
        }

        final int getEst() {
            int s; // force initialization
            final LinkedList<E> lst;
            if ((s = est) < 0) {
                if ((lst = list) == null)
                    s = est = 0;
                else {
                    expectedModCount = lst.modCount;
                    current = lst.first;
                    s = est = lst.size;
                }
            }
            return s;
        }

        public long estimateSize() { return (long) getEst(); }

        public Spliterator<E> trySplit() {
            Node<E> p;
            int s = getEst();
            if (s > 1 && (p = current) != null) {
                int n = batch + BATCH_UNIT;
                if (n > s)
                    n = s;
                if (n > MAX_BATCH)
                    n = MAX_BATCH;
                Object[] a = new Object[n];
                int j = 0;
                do { a[j++] = p.item; } while ((p = p.next) != null && j < n);
                current = p;
                batch = j;
                est = s - j;
                return Spliterators.spliterator(a, 0, j, Spliterator.ORDERED);
            }
            return null;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Node<E> p; int n;
            if (action == null) throw new NullPointerException();
            if ((n = getEst()) > 0 && (p = current) != null) {
                current = null;
                est = 0;
                do {
                    E e = p.item;
                    p = p.next;
                    action.accept(e);
                } while (p != null && --n > 0);
            }
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            Node<E> p;
            if (action == null) throw new NullPointerException();
            if (getEst() > 0 && (p = current) != null) {
                --est;
                E e = p.item;
                current = p.next;
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }

}
```