---
title: Thinking in Java - 学习笔记 - （十四）类型信息
date: 2018-05-02 14:52:39
categories:
	- 学习笔记
tags: 
	- Thinking in Java
	- Java
---


**运行时类型信息（RTTI）使得你可以在程序运行时发现和使用类型信息。**

它使你从只能在编译期执行面向类型的操作的禁锢中解脱了出来，并且可以使用某些非常强大的程序。
Java让我们在运行时识别对象和类的信息的方式：

- “传统的” **RTTI**，假定我们在编译时已经知道了所有的类型；

- “反射”机制，允许我们在运行时发现和使用类的信息。

<!-- more -->

为什么需要 **RTTI**
--

再次考虑基类 **Shape**，导出类 **Circle**、**Square**、**Triangle**。

把 **Shape** 对象放入 **List&lt;Shape&gt;** 的数组时会向上转型。但在向上转型为 **Shape** 的时候也丢失了对象的具体类型。对于数组而言，它们只是 **Shape** 类的对象。

当从数组中取出元素时，这种窗口——实际上它将所有的事物都当作 Object持有——会自动将结果转型回 Shape。

 > 这是 RTTI 最基本的使用形式，因为在 Java 中，所有的类型转换都是在运行时进行正确性检查。这也是 RTTI 名字的含义：在运行时，识别一个对象的类型。

在这里，RTTI 类型转换并不彻底：**Object** 被转型为 **Shape**，而不是转型为 **Circle** 等。这是因为目前我们只知道这个 **List&lt;Shape&gt;** 保存的是 **Shape**。在编译时，将由容器和 Java 的泛型系统来强制确保这一点；而在运行时，由类型转换操作来确保这一点。

接下来就是多态机制的事情了，**Shape** 对象实际执行什么样的代码，是由引用所指向的具体对象 **Circle**、**Square**、**Triangle** 决定的。

> 使用 **RTTI**，可以查询某个 **Shape** 引用所指向的对象的确切类型，然后选择或者剔除特例。

Class 对象
--

要理解 RTTI 在 Java 中的工作原理，首先必须知道类型信息在运行时是如何表示的。这项工作是由称为 Class <font face='kaiti'>对象</font> 的特殊对象完成的，它包含了与类有关的信息。

类是程序的一部分，每个类都有一个 Class 对象。换言之，每当编写并且编译了一个新类，就会产生一个 **Class** 对象（更恰当地说，是被保存在一个同名的 **.class** 文件中）。为了运行这个类的对象，运行这个程序的 Java 虚拟机（**JVM**）将使用被称为“类加载器”的子系统。

类加载器子系统实际上可以包含一条类加载器链，但是只有一个 <font face='kaiti'>原生类加载器</font>，它是 **JVM** 实现的一部分。原生类加载器加载的是所谓的 <font face='kaiti'>可信</font> 类，包括 Java API 类，它们通常是从本地盘加载的。在这条链中，通常不需要添加额外的类加载器，但是如果你有特殊需要，那么有一种方式可以挂接额外的类加载器。

所有的类都是在对其 *第一次* 使用时，动态加载到 **JVM** 中的。

当程序创建第一个对类的静态成员的引用时，就会加载这个类。这个证明构造器也是类的静态方法，即使在构造器之前并没有使用 **static** 关键字。

因此，Java 程序在它开始运行之前并非被完全加载，其各个部分是在必需时才加载的。

类加载器首先检查这个类的 **Class** 对象是否已经加载。如果尚未加载，默认的类加载器就会根据类名查找 **.class** 文件（例如，某个附加类加载器可能会在数据库中查找字节码）。在这个类的字节码被加载时，它们会接受验证，以确保其没有被破坏，并且不包含不良 Java 代码（这是 Java 中用于安全防范目的的措施之一）。

一旦某个类的 **Class** 对象被载入内存，它就被用来创建这个类的所有对象。

### Class 对象的方法

- forName(String) 输入全限定的类名，返回Class对象的引用

- isInterface()  是否为接口

- getInterfaces()  取得接口

- getSuperclass()  取得直接基类

- getSimpleName() 取得不含包名的类名。

- newInstance() 要含有默认构造器。

``` java

package thinkinginjava;

import java.util.Random;

class Initable {
    static final int staticFinal = 47;
    static final int staticFinal2 = ClassInitialization.rand.nextInt(1000);
    static {
        System.out.println("Initializing Initable");
    }
}

class Initable2 {
    static int staticNonFinal = 147;
    static {
        System.out.println("Initializing Initable2");
    }
}

class Initable3 {
    static int staticNonFinal = 74;
    static {
        System.out.println("Initiablizing Initable3");
    }
}

public class ClassInitialization {
    public static Random rand = new Random(47);

    public static void main(String[] args) {
        Class initable = Initable.class;
        System.out.println("After creating Initable ref");
        // 这行代码（staticFinal）不会触发初始化
        System.out.println(Initable.staticFinal);
        // 这行代码（staticFinal2）会触发初始化，因为它不是编译期常量
        System.out.println(Initable.staticFinal2);
        // 会触发初始化，因为不同时是final的
        System.out.println(Initable2.staticNonFinal);
        // 会触发初始化，因为要产生Class引用
        try {
//            System.out.println(Initable3.class.getName());
            Class initable3 = Class.forName("thinkinginjava.Initable3");
            System.out.println("After creating Initable3 ref");
            System.out.println(Initable3.staticNonFinal);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
} /* Output
After creating Initable ref
47
Initializing Initable
258
Initializing Initable2
147
Initiablizing Initable3
After creating Initable3 ref
74
*/
```


初始化有效地实现了尽可能的“惰性”。从对 **initable** 引用的创建中可以看到，仅使用 .class 语法来获得对类的引用不会引发初始化。但是，为了产生 **Class** 引用，**Class.forName()** 立即就进行了初始化。就像在对 **initable3** 引用的创建中所看到的。

如果一个 **static** 域不是 **final** 的，那么在对它访问时，问题要求在它被读取之前，要先进行链接（为这个域分配存储空间）和初始化（初始化该存储空间），就像在对 **Initable2.staticNonFinal** 的访问中所看到的那样。

> RTTI 在 Java 中还有第三种形式，就是关键字 **instanceof**。

instanceof 与 Class 的等价性
--

**instanceof** 和 **isInstance()** 保持了类型的概念，它指的是“你是这个类吗，或者你是这个类的派生类吗？”。而如果用 == 比较实际的 **Class** 对象，就没有考虑继承——它或者是这个确切的类型，或者不是。

反射：运行时的类信息
--

RTTI 和反射之间的真正的区别只在于，对 RTTI 来说，编译器在编译时打开和检查 **.class** 文件。而对于反射机制来说，**.class** 文件在编译时是不可获取的，所以是在运行时打开和检查 **.class** 文件。

动态代理
--

<font face='kaiti'>代理</font> 是基本的设计模式之一。
不是很懂，还要去看设计模式的书。

空对象
--

不必执行额外的对 **null** 的检查。

接口与类型信息
--

反射可以轻易地访问 **private** 方法，甚至匿名类。