---
title: Java学习笔记-数据结构-ArrayList
data: 2018-04-30 15:39:08
categories: 程序人生
tags: 
    - 数据结构与算法
    - Java
---
## 怎么理解ArrayList
一个实现List接口的可重置大小的数组。
## 构造函数
``` java
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
看一下涉及到的成员变量
transient 关键字先不去管它，我们看到，调用ArrayList的无参构造方法时，实际是把一个空的Object数组赋给了elementData。
<!--more-->
``` java
    transient Object[] elementData; // non-private to simplify nested class access

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

使用int类型的整数为ArrayList指定初始容量，分三种情况：
1. capacity > 0, 创建大小为capacity的Object数组。
2. capacity = 0, 创建一个空的Object数组。
3. capacity < 0, 抛出异常。

``` java
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

## add(E e)方法

先确保容量最小为 *size + 1*，保证下一个元素能够被加到ArrayList里。调用ensureCapacityInternal方法，确保新的元素能够放入ArrayList后，把元素放到表的末尾（紧跟在最后一个元素后面）。
``` java
    public boolean add(E e) {
        /*size 表示元素个数*/
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
考虑到使用无参构造函数ArrayList()的情况：

- 如果当前elementData为空数组，返回DEFAULT_CAPACITY与minCapacity中较大的一个
- 否则返回minCapacity
 
``` java
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
然后是ensureExplicitCapacity方法

``` java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;//fast-fail相关，记录对ArrayList结构修改的次数

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```
如果minCapacity - elementData.length > 0，也就是elementData数组装满了元素，那么就扩容为当前容量的1.5倍。

``` java
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

## add(int index, E e) 方法

与add(E e)的不同之处在于，add(int index, E e) 要把index及以后的元素后移一位。而且要先进行边界检查。
``` java
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


## remove的两个函数

``` java

    public E remove(int index) {
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


    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

```

