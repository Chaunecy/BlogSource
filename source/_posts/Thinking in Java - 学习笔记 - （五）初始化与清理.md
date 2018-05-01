---
title: Thinking in Java - 学习笔记 - （五）初始化与清理
date: 2018-05-01 18:20:38
tags:
	- Thinking in Java
	- Java
	- 学习笔记
---


涉及基本类型的重载
---------------------------

如何重载：

- 常数值被当作int值处理。

- 如果传入的实际参数类型*小于*方法中声明的形式参数类型，实际数据类型就会被提升。如果无法找到恰好接受char参数的方法，就会把char超脱提升至int型。

- 如果传入的实际参数*大于*重载方法声明的形式参数，就要通过类型转换来执行窄化转换。


### 为什么不以返回值区分重载方法

有时并不关心方法的返回值，想要的只是方法调用的其他效果（通常被称为“为了副作用面调用”），这时可能会调用方法而忽略其返回值。这时无法判断实际应该调用哪个方法。

``` java
    void f() {}
    int f() {return 1;}

    //应该执行哪一个函数？
    f();
```



this关键字
--------------

> this本身表示对当前对象的引用。

什么时候使用：

- 当需要返回对当前对象的引用时使用this。

- 将当前对象传递给其他方法时使用this。

- 在构造器中调用构造器。



static的含义
---------------

static方法就是没有this的方法。在static方法的内部不能调用非静态方法（并非完全不可能），反过来却可以。而且可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途。它很像全局方法。

> - 有些人认为static方法不是“面向对象”的，因为它们的确具有全局函数语义；
> 
> - 使用static方法时，由于不存在this，所以不是通过“向对象发送消息”的方式来完成的。


清理：终结处理和垃圾回收
------------------------------------

### finalize()方法
**特殊情况**：假定你的对象（并非使用new）获得了一块“特殊”的内存区域，由于垃圾回收器只知道释放那些经由new分配的内存，所以它不知道该如何释放该对象的这块“特殊”的内存。

**应对方法**：Java允许在类中定义一个名为finalize()的方法。它的工作原理“假定”是这样的：一旦垃圾回收器准备好释放对象占用的存储空间，将首先调用其finalize()方法，并且在下一次垃圾回收动作发生是，才会真正回收对象占用的内存。

> 在C++中，对象一定会被销毁（如果程序中没有缺陷的话）；而Java里的对象却并非总是被垃圾回收。或者：
>
> 1. 对象可能不被垃圾回收
> 2. 垃圾回收并不等于“析构”。

### finalize()的用途何在

finalize()不该被用作通用的清理方法，那么，它的真正用途是什么呢？
这引出了第三点：

> - 垃圾回收只与内存有关

也就是说，使用垃圾回收器的唯一原因是为了回收程序不再使用的内存。所以对于与垃圾回收有关的任何行为来说（尤其是finalize()方法），它们也必须同内存及其回收有关。

**疑问：这种特殊情况是怎么回事呢？**

之所以要有finalize()，是由于在分配内存时可能采用了类似C语言中的做法，而非Java中的通常做法。这种情况主要发生在使用“本地方法”的情况下。

### 一个示例程序

``` java
public class TerminationCondition {

    public static void main(String[] args) {
        Book novel = new Book(true, 1);
        novel.checkIn();
        int i = 0;
        for (; i < 0xaffff; i++) {
            new Book(true, i);
        }

        System.gc();
    }
}

class Book {
    boolean checkedOut = false;
    int isbn = 0;

    Book(boolean checkOut, int isbn) {
        checkedOut = checkOut;
        this.isbn = isbn;
    }

    void checkIn() {
        checkedOut = false;
    }

    protected void finalize() {
        if (checkedOut) {
            try {
                super.finalize();
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
            System.out.printf("Error: %-5x checked out\n", isbn);

        }
    }
}
```


垃圾回收器如何工作
--------------------------

### 垃圾回收机制

**垃圾回收技术**

1. <font face="kaiti" >引用记数</font>是一种简单但速度很慢的垃圾回收技术。每个对象都含有一个引用记数器，当有引用连接至对象时，引用计数加1。当引用离开作用域可被置为null时，引用计数减1。
> 缺陷：如果对象之间存在循环引用，可能会出现“对象应该被回收，但引用计数却不为零”的情况。

2. 自适应的垃圾回收技术。

  - 依据的思想：对任何“活”的对象，一定能最终追溯其到存活在堆栈或静态存储区之中的引用。

  - 一种做法：<font face="kaiti">停止-复制</font>（stop-and-copy）。

  - 切换模式：<font face="kaiti">标记-清扫</font>（mark-and-sweep）。没有新垃圾产生，转换到此模式。对一般用途而言，“标记-清扫”方式速度相当慢，但当知道只会产生少量垃圾甚至不会产生垃圾时，它的速度就很快。

     > 遍历所有引用，给存活的对象设一个标记，标记工作完成后，清理动 作都会开始。没有标记的对象被释放，清理后剩下的堆空间是不连续的。

成员初始化
--------------

Java尽力保证：所有变量在使用前都能得到恰当的初始化。
对于方法的局部变量，Java以编译时错误的形式来贯彻这种保证。类的数据成员（即字段）是基本类型，则保证每个基本类型数据成员都会有一个初始值。

### 静态数据的初始化

无论创建多少个对象，静态数据都只占用一份存储区域。static关键字不能应用于局部变量，因此它只能作用于域。

如果一个域是静态的基本类型域，且也没有对它进行初始化，那么它就会获得基本类型的标准初值；如果它是一个对象引用，那么它的默认初始化值就是null。

静态初始化只在必要时刻才会进行。只有在第一个对象被创建（或者第一次访问静态数据）的时候，它们才会被初始化。

被始化的顺序是先静态对象（如果它们尚未因前面的对象创建教程而被初始化），而后是“非静态”对象。

>总结一下对象的创建过程，假设有个名Dog的类：
>
> 1. 即使没有显式地使用static关键字，构造器实际上也是静态方法。因此，当首次创建类型为Dog的对象时（构造器可以看成静态方法），或者Dog类的静态方法/静态域首次被访问时，Java解释器必须查找类路径，以定位Dog.class文件。
> 
> 2. 然后载入Dog.class（这将创建一个Class对象），有关静态初始化的所有动作都会执行。因此，静态初始化只在Class对象首次加载的时候进行一次。
> 
> 3. 当用new Dog()创建对象的时候，首先将在堆上为Dog对象分配足够的存储空间。
> 
> 4. 这块存储空间会被清零，这就自动地将Dog对象中的所有基本类型数据都设置成了默认值（对数字来说是0，对布尔型和字符型也相同），而引用则被设置成了null。
> 
> 5. 执行所有出现于字段定义处的初始化动作。
> 
> 6. 执行构造器。正如将在第7章所看到的，这可能会牵涉到很多动作，尤其是涉及继承的时候。


``` java
public class TestStatic {

    public int method() {
        //static int i = 0;    static 关键字不能在方法中使用
        return 0;

    }
    public static void main(String[] args) {
        Dog dog = new Dog(1);
        Dog doggy = new Dog(2);
    }
}

```

``` java
class Dog {
    private int id;

    static {
        System.out.println("static 字段");
    }

    {
        System.out.printf("dog id: %-2d\n",id);
    }

    Dog(int id) {
        this.id = id;
        System.out.println("Dog(int) 构造器， id = " + id);
    }
}

```

``` markdown
输出结果：
static 字段                 //只会初始化一次
dog id: 0                  // 创建对象时都会初始化
Dog(int) 构造器， id = 1
dog id: 0                  // 创建对象时都会初始化
Dog(int) 构造器， id = 2
```

数组初始化
--------------

### 可变参数列表

可变参数列表不依赖于自动包装机制，而实际上使用的是基本类型。
``` java
package TIK;

class NewVarArgs {
    private static void printArray(Object... args) {
        for (Object obj : args) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
    private static class A {

    }
    public static void main(String[] args) {
        printArray(47, 3.14, 11.11);
        printArray(new A(), new A(), new A());
        printArray((Object) new int[]{1, 3, 2, 4});
    }
}/*
输出结果:
47 3.14 11.11 
TIK.NewVarArgs$A@1540e19d TIK.NewVarArgs$A@677327b6 TIK.NewVarArgs$A@14ae5a5 
[I@7f31245a
*/
```

你应该总是只在重载方法的一个版本上使用可变参数列表，或者压根就不使用它。

``` java
    private static void f(float i, Character... chars) {

    }
    private static void f(Character... chars) {

    }

    public static void main(String[] args) {
        f(1, 'a');
        f('a', 'b');
    }
/*
Error:(23, 9) java: 对f的引用不明确
  TIK.NewVarArgs 中的方法 f(float,java.lang.Character...) 和 TIK.NewVarArgs 中的方法 f(java.lang.Character...) 都匹配
*/
```


枚举类型
--------------
见后面章节

