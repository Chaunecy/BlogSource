---
title: ten
date: 2018-05-01 19:11:59
tags:
	- Thinking in Java
	- Java
	- 学习笔记
---

**可以将一个类的定义放在另一个类的内部，这就是内部类。**

内部类是一种非常有用的特性，因为它允许你把一些逻辑相关的类组织在一起，并控制位于内部的类的可视性。然而，内部类与组合是完全不同的概念。

链接到外部类
---

当生成一个内部类的对象时，与制造它的<font face="kaiti">外围对象</font>（enclosing object)之间就有了一种联系，所以它能访问其外围对象的所有成员，而<font face="kaiti">不需要</font>任何特殊条件。此外，内部类还拥有其外围类的所有元素的访问权。

> 当某个外围类的对象创建了一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外围类对象的引用。然后，在访问此外围类的成员时，就是用那个引用来选择外围类的成员。（内部类非**static**）


使用.this与.new
--

在拥有外部类对象之前是不可能创建内部类对象的。这是因为内部类对象会暗暗地连接到创建它的外部类对象上。但是如果创建的是<font face="kaiti">嵌套类</font>（静态内部类），那么它就不需要对外部类对象的引用。

``` java
// .new
public class DotNew {

    public class Inner {
    }

    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

``` java
// .this
public class DotThis {
    void f() {
        System.out.println("DotThis.f()");
    }

    public class Inner {
        public DotThis outer() {
            return DotThis.this;
            // A plain "this" would be Inner's "this"
        }
    }

    public Inner inner() {
        return new Inner();
    }

    public static void main(String[] args) {
        DotThis dt = new DotThis();
        DotThis.Inner dti = dt.inner();
        dti.outer().f();
    }
}

```

内部类与向上转型
---

当将身上转型为其基类，尤其是转型为一个接口的时候，内部类就有了用武之地。（从实现了某个接口的对象，得到对此接口的引用，与向上转型为这个对象的基类，实质上效果是一样的。）这是因为此内部类——某个接口的实现——能够完全不可见，并且不可用。所得到的只是指向基类或接口的引用，所以能够很方便地隐藏实现细节。

``` java

interface Destination {
    String readLabel();
}

interface Contents {
    int value();
}

class Parcel4 {
    private class PContents implements Contents {
        private int i = 11;

        @Override
        public int value() {
            return i;
        }
    }

    protected class PDestination implements Destination {
        private String label;

        private PDestination(String whereTo) {
            label = whereTo;
        }

        @Override
        public String readLabel() {
            return label;
        }
    }

    public Destination destination(String s) {
        return new PDestination(s);
    }

    public Contents contents() {
        return new PContents();
    }
}

public class TestParcel {
    public static void main(String[] args) {
        Parcel4 p = new Parcel4();
        Contents c = p.contents();
        Destination d = p.destination("Tasmania");
        // Illegal -- can't access private class
        //! Parcel4.PContents pc = p.new PContents();
    }
}

```
在方法和作用域的内部类
---

可以在任意的作用域创建内部类（类的创建是没有条件的，它其实与别的类一起编译过了）；在作用域外不可用，除此之外与普通的类一样。

匿名内部类
---

``` java
class Wrapping {
    private int i;

    public Wrapping(int x) {
        i = x;
    }

    public int value() {
        return i;
    }
}

public class TestParcel {

    public Wrapping wrapping(int x) {
        return new Wrapping(x) {
            public int value() {
                return super.value() * 47;
            }
        };
    }

    public static void main(String[] args) {
        TestParcel p = new TestParcel();
        Wrapping w = p.wrapping(10);
        System.out.printf("w.value(): %d\n",w.value());
    }
}
```

### 再访工厂方法

``` java
interface Service {
    void method1();

    void method2();
}

interface ServiceFactory {
    Service getService();
}

class Implementation1 implements Service {

    @Override
    public void method1() {
        System.out.println(getClass().getSimpleName() + ": method1");
    }

    @Override
    public void method2() {
        System.out.println(getClass().getSimpleName() + ": method2");
    }
    // jdk1.8
    public static ServiceFactory factory = Implementation1::new;
}


class Implementation2 implements Service {

    @Override
    public void method1() {
        System.out.println(getClass().getSimpleName() + ": method1");
    }

    @Override
    public void method2() {
        System.out.println(getClass().getSimpleName() + ": method2");
    }
    // jdk1.8
    public static ServiceFactory factory = Implementation2::new;
}

public class Factories {
    public static void serviceConsumer(ServiceFactory fact) {
        Service s = fact.getService();
        s.method1();
        s.method2();
    }

    public static void main(String[] args) {
        serviceConsumer(Implementation1.factory);
        serviceConsumer(Implementation2.factory);
    }
}
```
嵌套类
---

如果不需要内部类对象与其外围类对象之间有联系，那么可以将内部类声明为**static**。这通常称为<font face="kaiti">嵌套类</font>。想要理解**static**应用于内部类时的含义，就必须记住，普通的内部类对象隐式地保存了一个引用，指向创建它的外围类对象。然而当内部类是**static**的时，就不是这样了。

嵌套类意味着：

1. 要创建嵌套类的对象，并不需要其外围类的对象。

2. 不能从嵌套类的对象中访问非静态的外围类对象。


嵌套类与普通的内部类还有一个区别。普通内部类的字段与方法，只能放在类的外部层次上，所以普通的内部类不能有**static**数据和**static**字段，也不能包含嵌套类。但是嵌套类可以包含所有这些东西。

### 接口内部的类

正常情况下，不能在接口内部放置任何代码，但嵌套类可以作为接口的一部分。放到接口跌任何类都自动是**public**和**static**的。因为类是**static**的，只是将嵌套类置于接口的命名空间内，这并不违反接口的规则。甚至可以在内部类中实现其外围接口，如果想要创建某些公共代码，使得它们可以被某个接口的所有不同实现所共用，那么使用接口内部的嵌套类会显得很方便。

### 从多层嵌套类中访问外部类的成员

一个内部类被嵌套多少层并不重要——它能透明地访问所有它所嵌入的外围类的所有成员。

可以看到在M.A.B中，调用方法g()和f()不需要任何条件（即使它们被定义为**private**）。**“.new”**语法能产生正确的作用域，所以不必在调用构造器时限定类名。

``` java
class M {
    private void f() {}
    class A {
        private void g() {}
        public class B {
            void h() {
                g();
                f();
            }
        }
    }
}

public class MultiNestingAccess {
    public static void main(String[] args) {
        M m = new M();
        M.A ma = m.new A();
        M.A.B mab = ma.new B();
        mab.h();
    }
}
```

为什么需要内部类
---

一般说来，内部类继承处某个类或实现某个接口，内部类的代码操作创建它的外围类的对象。所以可以认为内部类提供了某种进入其外围类的窗口。

内部类必须要回答的一个问题是：如果只是需要一个对接口的引用，为什么不通过外围类实现那个接口呢？

答案是：“如果这能满足需求，那么就应该这样做。”那么内部类实现一个接口与外围类实现这个接口有什么区别呢？答案是：后者不是总能享用到接口带来的方便，有时需要乃至接口的实现。所以使用内部类最吸引人的原因是：

<font face="kaiti">每个内部类才能独立地继承处一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。</font>

### 闭包与回调

<font face="kaiti">闭包</font>（closure）是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。通过这个定义，可以看出内部类是面向对象的闭包，因为它不仅包含外围类的对象（创建内部类的作用域）的信息，还自动拥有一个指向此外围类对象的引用，在此作用域内，内部类有权操作所有的成员，包括**private**成员。

### 内部类与控制框架（control framework）

<font face="kaiti">应用程序框架</font>（application framework）就是被设计用以解决某类特定问题的一个类或一组类。
控制框架是一类特殊的应用程序框架，它用来解决响应事件的需求。

``` java
public class Controller {
    // A class from java.util to hold Event objects:
    private List<Event> eventList = new ArrayList<Event>();
    public void addEvent(Event c) { eventList.add(c); }
    public void run() {
    while(eventList.size() > 0) {
        // Make a copy so you're not modifying the list
        // while you're selecting the elements in it
        for(Event e : new ArrayList<Event>(eventList))
            if(e.ready()) {
                System.out.println(e);
                e.action();
                eventList.remove(e);
            }
    }
}
```

run()方法循环遍历eventList，寻找就绪的（ready()）、要运行的Event对象。对找到的每一个就绪的事件，使用对象的toString()打印其信息，调用其action()方法，然后从队列中移除此Event。

注意，在目前的设计中你并不知道Event到底<font face="kaiti">做了什么</font>。这正是此设计的关键所在，“使变化的事物与不变的事物相互分离”。

这正是内部类要做的事情，内部类允许：

: 1）控制框架的完整实现是由单个的类创建的，从而使得实现的细节被封装了起来。内部类用来表示解决问题所必需的各种不同的**action()**。

: 2）内部类能够很容易地访问外围类的任意成员，所以可以避免这种实现变得笨拙。



内部类的继承
---

``` java
class WithInner {
    class Inner {}
}

public class InheritInner extends WithInner.Inner {
    //! InheritInner() {} // Won't compile
    InheritInner(WithInner wi) {
        wi.super();
    }
    public static void main(String[] args) {
        WithInner wi = new WithInner();
        InheritInner ii = new InheritInner(wi);
    }
}

```

内部类可以被继承吗？
---

当继承了某个外围类的时候，内部类并没有发生什么特别神奇的变化。这两个内部类是完全独立的两个实体，各自在自己的命名空间内。明确地继承某个内部类是可以的。

局部内部类
---
使用局部内部类的理由：

1. 需要一个已命名的构造器，或者需要重载构造器，而匿名内部类只能用于实例初始化。

2. 需要不止一个该内部类的对象。


内部类标识符
---

内部类也必须生成一个**.class**文件以包含它们的**Class**对象信息。这些类文件的命名有严格的规则：外围类的名字，加上“**$**”，再加上内部类的名字。

如果内部类是匿名的，编译器会简单地产生一个数字作为其标识符。

总结
--

接口、内部类与多态机制等特性的使用应该是设计阶段来考虑的问题。

