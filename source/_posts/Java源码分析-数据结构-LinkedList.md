---
title: Java源码分析 - 数据结构 - LinkedList
date: 2018-05-21 12:19:13
categories: 
	- 源码分析
tags: 
    - 数据结构与算法
    - Java
---

## 什么是LinkedList

> Doubly-linked list implementation of the {@code List} and {@code Deque} interfaces.  Implements all optional list operations, and permits all elements (including {@code null}).

实现 **List** 和 **Deque** 接口的双向链表，允许存入空值。**LinkedList** 不保证线程安全。

<!-- more -->

## 构造函数

### 无参构造函数

``` java
	public LinkedList() {
    }
```

### 使用已有容器的构造函数

``` java
	public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c); // 实现见下方
    }
```

## 添加元素：增



### 添加一个元素到末尾

``` java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null) // 添加第一个元素
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

### 添加一个元素到指定位置

``` java
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }

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

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```

其中node()函数的实现为

``` java
    Node<E> node(int index) {
        // assert isElementIndex(index);
        // 离哪头近，就从哪头开始找，但还是很花时间
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
```

### 添加给定的容器内的元素

``` java
    // 从链表最后添加新的元素
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
    // size 表示元素数
    transient int size = 0;
    // 从指定位置添加新的元素
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index); //范围检查

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        // 放到链表最后
        if (index == size) {
            succ = null;
            pred = last; // last: 最后一个元素
        } else { // 插入到链表中间
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null) // 尚未初始化
                first = newNode; // 链表的头节点，指向第一个元素
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred; // 链表的尾节点，指向最后一个元素
        } else {
            pred.next = succ; // 善后工作，所有元素添加完成，
            succ.prev = pred; // 把新元素与旧有元素链接在一起。
        }

        size += numNew;
        modCount++;
        return true;
    }
```

