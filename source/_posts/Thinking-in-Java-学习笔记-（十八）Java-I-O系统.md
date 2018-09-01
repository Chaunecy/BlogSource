---
title: Thinking in Java - 学习笔记 - （十八）Java I/O系统
date: 2018-05-02 15:00:10
categories:
	- 学习笔记
tags: 
	- Thinking in Java
	- Java
---

File 类
---

它既能代表一个特定文件的名称，又能代表一个目录下的一组文件的名称。

输入和输出
--

Java 类库中的 I/O 类分成输入和输出两部分。

nio
--

速度的提高来自于所使用的结构更接近于操作系统执行 I/O 的方式：通道和缓冲器。

对象序列化
--

Java 的对象序列化将那些实现了 **Serializable** 接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。

**transient** 关键字修饰的字段不会被序列化。
