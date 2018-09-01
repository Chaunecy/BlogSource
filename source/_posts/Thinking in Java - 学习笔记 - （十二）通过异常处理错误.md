---
title: Thinking in Java - 学习笔记 - （十二）通过异常处理错误
date: 2018-05-02 14:23:14
categories:
	- 学习笔记
tags: 
	- Thinking in Java
	- Java
---

**Java的基本理念是结构不佳的代码不能运行。**

发现错误的理想时机是在编译阶段。有的错误必须在运行期间解决。
改进的错误恢复机制是提供代码健壮性的最强有力的方式。

使用异常往往能够降低错误处理代码的复杂度。把“描述在正常执行过程中做什么事”的代码和”出了问题怎么办”的代码相分离。

<!-- more -->

基本异常
---

<font face="kaiti">异常情形</font>（exceptional condition）是指阻止当前方法或者作用域继续执行的问题。对于异常情形，因为在<font face="kaiti">当前环境</font>下无法获得必要的信息来解决问题，只能从当前环境跳出，并把问题提交给上一级环境。

当抛出异常后，有几件事会随之发生。

- 首先，同Java中其他对象的创建一样，将使用**new**在堆上创建异常对象。

- 然后，当前的执行路径（它不能继续下去了）被终止，并且从当前环境中弹出对异常对象的引用。

- 异常处理机制接管程序，并开始寻找一个恰当的地方来继续执行程序。

 > 这个恰当的地方就是<font face="kaiti">异常处理程序</font>，它的任务是将程序从错误状态中恢复，以使程序能要么换一种方式运行，要么继续运行下去。

异常使得我们可以将每件事都当作一个事务来考虑，而异常可以看护这些事务的底线。我们还可以将异常看作是一种内建的恢复（undo）系统，因为（在细心使用的情况下）把我们在程序中可以拥有各种不同的恢复点。

异常最重要的方面之一就是如果发生问题，它们将不允许程序沿着其正常的路径继续走下去。

### 终止与恢复

异常处理理论上有两种基本模型：<font face="kaiti">终止模型</font>和<font face="kaiti">恢复模型</font>

> 恢复模型不是很实用，其中的主要原因可能是它所导致的耦合：恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于招聘位置的非通用性代码。

异常声明
---

``` java
    void f() throws TooBig, TooSmall, DivZero { //...
```
捕获所有异常
---

### 栈轨迹

``` java
public class WhoCalled {
    static void f() {
        // Generate an exception to fill in the stack trace
        try {
            throw new Exception();
        } catch (Exception e) {
            for (StackTraceElement item : e.getStackTrace()) {
                System.out.println(item.getMethodName());
            }
        }
    }
    static void g() { f(); }
    static void h() { g(); }

    public static void main(String[] args) {
        f();
        System.out.println("---------------------");
        g();
        System.out.println("---------------------");
        h();
    }
} /*
输出结果
f
main
---------------------
f
g
main
---------------------
f
g
h
main
*/
```

Java标准异常
----

### RuntimeException

``` java
    if (ref == null)
        throw new NullPointerExcception;
         
```

如果对**null**进行调用，Java会自动抛出**NullPointerException**异常，所以上述写法是不必要的。

只能在代码中忽略**RuntimeException**（及其子类）类型的异常，其他类型的异常的处理都是由编译器强制实施的。究其原因，**RuntimeException**代表的是编程错误：

- 无法预料的错误，如**null**引用

- 作为程序员，应该在代码中进行检查的错误

使用finally进行清理
---
对于没有垃圾回收和析构函数自动调用机制的语言来说，**finally**非常重要。它能使程序员保证：无论**try**块里发生了什么，内存总能得到释放。但Java有垃圾回收机制，所以内存释放不再是问题。而且，Java也没有析构函数可供调用。那么Java在什么情况下才能用到**finally**呢？

当要把除内存之外的资源恢复到它们的初始状态时，就要用到**finally**子句。这种需要清理的资源包括：已经打开的文件或网络连接，在屏幕上画的图形等。

当涉及**break**和**continue**语句的时候，**finally**子句也会得到执行。**return**中也可以保证执行。

### 缺憾：异常丢失

``` java
    public static void main(String[] args) {
        try {
            throw new RuntimeException();
        }
        finally {
            // 不产生任何输出
            return;
        }
    }
/*
注释掉 return 一行后
Exception in thread "main" java.lang.RuntimeException
	at thinkinginjava.WhoCalled.main(WhoCalled.java:19)
*/
```


异常的限制
----

当覆盖方法的时候，只能抛出在基类方法的异常说明里列出的那些异常。

构造器
---

对于在构造阶段可能会抛出异常，并且要求清理的类，最安全的使用方式是使用嵌套的**try**子句：

``` java
public class Cleanup {
    public static void main(String[] args) {
        try {
            InputFile in = new InputFile("Cleanup.java");
            try {
                String s;
                int i = 1;
                while((s = in.getLine()) != null)
                    // perform line-by-line processing here
            } catch(Exception e) {
                
            } fanilly {
                in.dispose();
            }
        } catch(Exception e) {
            System.out.println("InputFile construction failed");
        }
    }
}
```

异常处理的一个重要原则是“只有在你知道如何处理的情况下才捕获异常”。实际上，异常处理的一个重要目标就是把错误处理的代码同错误发生的地点相分离。

### “被检查的异常”：
如果不知道怎么处理异常，可以使用异常链，把“被检查的异常”包装进**RuntimeException**里面。