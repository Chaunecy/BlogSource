---
title: Java学习笔记 - 数据结构 - ArrayList
date: 2018-04-30 15:39:08
categories: 
	- 源码分析
tags: 
	- 数据结构与算法
	- Java
---
## 怎么理解 ArrayList

一个实现 **List** 接口的可扩展的数组（Resizable-array）。与 **Vector** 大致上相同，但是不保证线程安全。

## 构造函数

### 无参构造函数

```java
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

看一下涉及到的成员变量
**transient** 关键字先不去管它，我们看到，调用**ArrayList**的无参构造方法时，实际是把一个空的**Object**数组赋给了**elementData**。
<!--more-->

```java
    transient Object[] elementData; // non-private to simplify nested class access

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

### 指定容量的构造函数

使用int类型的整数为**ArrayList**指定初始容量，分三种情况：

1. **capacity** > 0, 创建大小为capacity的Object数组。
2. **capacity** = 0, 创建一个空的Object数组。
3. **capacity** < 0, 抛出异常。

```java
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                                   initialCapacity);
        }
    }
```

### 使用已有容器的构造函数

```java
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

```

## 插入元素：增

### 在列表末尾插入元素

先确保容量最小为 *size + 1*，保证下一个元素能够被加到**ArrayList**里。

调用**ensureCapacityInternal**方法，确保新的元素能够放入ArrayList。

最后把元素放到末尾（紧跟在最后一个元素后面）。

```java
    public boolean add(E e) {
        /*size 表示元素个数*/
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

考虑到使用无参构造函数**ArrayList()**的情况：

- 如果当前**elementData**为空数组，返回**DEFAULT_CAPACITY**与**minCapacity**中较大的一个
- 否则返回**minCapacity**

```java
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

然后是**ensureExplicitCapacity**方法

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;//fast-fail相关，记录对ArrayList结构修改的次数

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

如果**minCapacity - elementData.length > 0**，也就是**elementData**数组装满了元素，那么就扩容为当前容量的**1.5**倍。

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        /* 扩大为当前容量的1.5倍*/
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        /* 如果还是不够，把容量设置为minCapacity的值*/
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        /*
         * 但是这样有可能溢出，那么就和 MAX_ARRAY_SIZE 比较一下
         * 如果确实比 MAX_ARRAY_SIZE大，调用hugeCapacity函数
        */
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        /* 创建一个新的数组，数组长度为newCapacity，把元素复制到新的数组 */
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    /* 某些VM会保留一些头部字(header words) */
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /* 允许的最大数组大小 */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

### 在指定位置插入元素

与**add(E e)**的不同之处在于，**add(int index, E e)** 要把**index**及以后的元素后移一位。而且要先进行边界检查。

```java
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

## 移除元素：删

### 按索引删除

```java
    public E remove(int index) {
        // 是否越界
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    // index为负数时会抛数组越界异常，在elementData()函数。
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        // 这里进行了强制类型转换
        // 因为Java的泛型会erase实际类型，所以无法创建泛型数组。
        // 这里就用了Object数组保存元素，获取时进行类型转换
        return (E) elementData[index];
    }
```

### 按元素删除

```java
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
                    // 删除第一个符合条件的
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }


    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

## 替换元素：改

```java
    public E set(int index, E element) {
        rangeCheck(index);
        //方法名与字段名取一样的不会很难受吗？
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

## 检索元素：查

### 获取对应下标的元素

```java
	public E get(int index) {
        rangeCheck(index);
        
        return elementData(index);
    }
```

### 是否存在某元素

```java
    // 是否存在此元素
	public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

### 获取某元素的下标

``` java
    // 从前往后查询。返回元素的下标，-1表示没有。
    // ！！equals方法要自己实现即Override，不然默认比较对象的地址。
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
```

### 获取某元素的下标 - 反向查找

``` java
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

```

## 迭代器

### 只能正向遍历

``` java
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return 指向下一个！！！
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount; // 不允许更改

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }
        // 用于遍历
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
        // 在这里删除
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

        // 应该是观察者模式用的吧？
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
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
        // 不允许在迭代过程中进行非预期的插入、删除等结构性的修改
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

### 更多功能

向前、向后，增加、修改。

``` java
public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

	public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

	private class ListItr extends Itr implements ListIterator<E> {
        // 指定从哪里开始迭代
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
```

