---
title: Java源码分析 - ReentrantLock
date: 2018-05-08 09:31:56
tags:
	- Java
	- 并发
	- 源码分析
---





## javadoc说明

> A reentrant mutual exclusion {@link Lock} with the same basic behavior and semantics as the implicit monitor lock accessed using {@code synchronized} methods and statements, but with extended capabilities.


可重入锁：一个可重入的互相排斥的锁，和隐式地使用**synchronized**方法获得的锁有着相同的基本行为与语义。

## 加锁

### 函数说明

如果锁没有被另一个线程占有，则立即返回，并把锁的计数设为1。

如果当前线程已经获取到

### 代码

``` java
    public void lock() {
        sync.lock();
    }
```




