---
title: Java容器ArrayList源码完整注释
tags: [Java, 源码分析, 概念原理]
---

本文源码基于JDK8
{:.info}

## ArrayList类继承关系

![image-20220629170718914](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/202206291707130.png)

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
                // 扩容后长度大于数组最大安全长度的处理，调用hugeCapacity方法
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

## RandomAccess接口

```java
// 标记式接口，支持随机访问
public interface RandomAccess {
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

## ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    // 初始容量
    private static final int DEFAULT_CAPACITY = 10;

    // 显式指定长度为0，使用此共享空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // 使用默认长度，在未加入元素时，暂时使用此共享空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // 真实元素内容存放的地方
    transient Object[] elementData;

    // ArrayList元素个数
    private int size;

    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // 按初始容量构造elementData对象数据
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // 显示指定长度为0，使用共享空数组EMPTY_ELEMENTDATA
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

    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                // 参数c是ArrayList类型，直接使用toArray转换出来的数组
                elementData = a;
            } else {
                // 参数c不是ArrayList类型，再复制一次数组
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // 没有元素，使用共享空数组
            elementData = EMPTY_ELEMENTDATA;
        }
    }

    // 缩减elementData多余空间
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

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

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 假如elementData是DEFAULTCAPACITY_EMPTY_ELEMENTDATA空数组，则至少扩容到DEFAULT_CAPACITY
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 当前要求的最小容量比elementData长度要大，需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    // 数组元素长度安全的最大长度
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 扩充elementData
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 常规扩充方法
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            // 还不够的话按要求的最小量来扩充
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }

    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        // 直接通过数组访问需要的元素
        return (E) elementData[index];
    }

    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    public boolean add(E e) {
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);
        // 插入的目标位置后面的所有元素全部向后移动一个位置
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        // 删除操作实际需要移动的元素个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null;

        return oldValue;
    }

    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    // 内部快速删除元素方法，不检查index是否越界，不返回删除的元素值
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null;
    }

    public void clear() {
        modCount++;

        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        // 确保长度足够，把需要增加的所有元素附加到elementData尾部
        ensureCapacityInternal(size + numNew);
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }

    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            if (r != size) {
                // 假如抛异常，即elementData未遍历完成，将当前正在遍历的和未遍历的元素全部当作保留下来处理
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                // 删元素的次数加到modCount
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        int expectedModCount = modCount;
        s.defaultWriteObject();

        s.writeInt(size);

        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        s.defaultReadObject();

        s.readInt();

        if (size > 0) {
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }

    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        int cursor;
        int lastRet = -1;
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
    }

    // 不同于AbstractList，这里SubList是个内部类
    private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;

        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }

        public E set(int index, E e) {
            rangeCheck(index);
            checkForComodification();
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }

        public int size() {
            checkForComodification();
            return this.size;
        }

        public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }

        public E remove(int index) {
            rangeCheck(index);
            checkForComodification();
            E result = parent.remove(parentOffset + index);
            this.modCount = parent.modCount;
            this.size--;
            return result;
        }

        protected void removeRange(int fromIndex, int toIndex) {
            checkForComodification();
            parent.removeRange(parentOffset + fromIndex,
                               parentOffset + toIndex);
            this.modCount = parent.modCount;
            this.size -= toIndex - fromIndex;
        }

        public boolean addAll(Collection<? extends E> c) {
            return addAll(this.size, c);
        }

        public boolean addAll(int index, Collection<? extends E> c) {
            rangeCheckForAdd(index);
            int cSize = c.size();
            if (cSize==0)
                return false;

            checkForComodification();
            parent.addAll(parentOffset + index, c);
            this.modCount = parent.modCount;
            this.size += cSize;
            return true;
        }

        public Iterator<E> iterator() {
            return listIterator();
        }

        public ListIterator<E> listIterator(final int index) {
            checkForComodification();
            rangeCheckForAdd(index);
            final int offset = this.offset;

            return new ListIterator<E>() {
                int cursor = index;
                int lastRet = -1;
                int expectedModCount = ArrayList.this.modCount;

                public boolean hasNext() {
                    return cursor != SubList.this.size;
                }

                @SuppressWarnings("unchecked")
                public E next() {
                    checkForComodification();
                    int i = cursor;
                    if (i >= SubList.this.size)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i + 1;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public boolean hasPrevious() {
                    return cursor != 0;
                }

                @SuppressWarnings("unchecked")
                public E previous() {
                    checkForComodification();
                    int i = cursor - 1;
                    if (i < 0)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i;
                    return (E) elementData[offset + (lastRet = i)];
                }

                @SuppressWarnings("unchecked")
                public void forEachRemaining(Consumer<? super E> consumer) {
                    Objects.requireNonNull(consumer);
                    final int size = SubList.this.size;
                    int i = cursor;
                    if (i >= size) {
                        return;
                    }
                    final Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length) {
                        throw new ConcurrentModificationException();
                    }
                    while (i != size && modCount == expectedModCount) {
                        consumer.accept((E) elementData[offset + (i++)]);
                    }
                    // update once at end of iteration to reduce heap write traffic
                    lastRet = cursor = i;
                    checkForComodification();
                }

                public int nextIndex() {
                    return cursor;
                }

                public int previousIndex() {
                    return cursor - 1;
                }

                public void remove() {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        SubList.this.remove(lastRet);
                        cursor = lastRet;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void set(E e) {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        ArrayList.this.set(offset + lastRet, e);
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void add(E e) {
                    checkForComodification();

                    try {
                        int i = cursor;
                        SubList.this.add(i, e);
                        cursor = i + 1;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (expectedModCount != ArrayList.this.modCount)
                        throw new ConcurrentModificationException();
                }
            };
        }

        
        public List<E> subList(int fromIndex, int toIndex) {
            subListRangeCheck(fromIndex, toIndex, size);
            return new SubList(this, offset, fromIndex, toIndex);
        }

        private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+this.size;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }

        public Spliterator<E> spliterator() {
            checkForComodification();
            return new ArrayListSpliterator<E>(ArrayList.this, offset,
                                               offset + this.size, this.modCount);
        }
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

	// 重写了AbstractList的实现
    @Override
    public Spliterator<E> spliterator() {
        return new ArrayListSpliterator<>(this, 0, -1, 0);
    }

    // 跳过Spliterator的分析
    /** Index-based split-by-two, lazily initialized Spliterator */
    static final class ArrayListSpliterator<E> implements Spliterator<E> {

        /*
         * If ArrayLists were immutable, or structurally immutable (no
         * adds, removes, etc), we could implement their spliterators
         * with Arrays.spliterator. Instead we detect as much
         * interference during traversal as practical without
         * sacrificing much performance. We rely primarily on
         * modCounts. These are not guaranteed to detect concurrency
         * violations, and are sometimes overly conservative about
         * within-thread interference, but detect enough problems to
         * be worthwhile in practice. To carry this out, we (1) lazily
         * initialize fence and expectedModCount until the latest
         * point that we need to commit to the state we are checking
         * against; thus improving precision.  (This doesn't apply to
         * SubLists, that create spliterators with current non-lazy
         * values).  (2) We perform only a single
         * ConcurrentModificationException check at the end of forEach
         * (the most performance-sensitive method). When using forEach
         * (as opposed to iterators), we can normally only detect
         * interference after actions, not before. Further
         * CME-triggering checks apply to all other possible
         * violations of assumptions for example null or too-small
         * elementData array given its size(), that could only have
         * occurred due to interference.  This allows the inner loop
         * of forEach to run without any further checks, and
         * simplifies lambda-resolution. While this does entail a
         * number of checks, note that in the common case of
         * list.stream().forEach(a), no checks or other computation
         * occur anywhere other than inside forEach itself.  The other
         * less-often-used methods cannot take advantage of most of
         * these streamlinings.
         */

        private final ArrayList<E> list;
        private int index; // current index, modified on advance/split
        private int fence; // -1 until used; then one past last index
        private int expectedModCount; // initialized when fence set

        /** Create new spliterator covering the given  range */
        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; // OK if null unless traversed
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }

        private int getFence() { // initialize fence to size on first use
            int hi; // (a specialized variant appears in method forEach)
            ArrayList<E> lst;
            if ((hi = fence) < 0) {
                if ((lst = list) == null)
                    hi = fence = 0;
                else {
                    expectedModCount = lst.modCount;
                    hi = fence = lst.size;
                }
            }
            return hi;
        }

        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
                new ArrayListSpliterator<E>(list, lo, index = mid,
                                            expectedModCount);
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            if (action == null)
                throw new NullPointerException();
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi, mc; // hoist accesses and checks from loop
            ArrayList<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            if ((lst = list) != null && (a = lst.elementData) != null) {
                if ((hi = fence) < 0) {
                    mc = lst.modCount;
                    hi = lst.size;
                }
                else
                    mc = expectedModCount;
                if ((i = index) >= 0 && (index = hi) <= a.length) {
                    for (; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) a[i];
                        action.accept(e);
                    }
                    if (lst.modCount == mc)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }

        public long estimateSize() {
            return (long) (getFence() - index);
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }

    @Override
    public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                // 假如第i个元素符合条件，则把removeSet中的对应位设置为true，标记这个元素需要被删除
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            // 将没被删除的元素移动到ArrayList的最左边
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                // 从当前位置开始，获取下一个标记为false的位置，也就是下一个不被删除的元素位置
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }

    @Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = this.size;
        // 相对于List接口的默认实现，遍历方式从ListIterator改为了数组for循环遍历
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    @Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
}
```