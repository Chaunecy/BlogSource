---
title: Thinking in Java - 学习笔记 - （十五）泛型
date: 2018-05-02 14:56:22
categories:
	- 学习笔记
tags: 
	- Thinking in Java
	- Java
---

泛型实现了 <font face='kaiti'>参数化类型</font> 的概念。“泛型”这个术语的意思是：“适用于许多许多的类型”。


泛型方法
---

无论何时，只要你能做到，你就应该尽量使用泛型方法。也就是说，如果使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型方法，因为它可以使事情更清楚明白。另外，对于一个 **static** 的方法而言，无法访问泛型类的类型参数，所以，如果 **static** 方法需要使用泛型能力，就必须使其成为泛型方法。

<!-- more -->

``` java
package generics;

public class GenericMethods {
    public <T> void f(T x) {
        System.out.println(x.getClass().getName());
    }

    public static void main(String[] args) {
        GenericMethods gm = new GenericMethods();
        gm.f("");
        gm.f(1);
        gm.f(.0);
        gm.f(.0f);
        gm.f('c');
        gm.f(gm);
    }
} /* Output
java.lang.String
java.lang.Integer
java.lang.Double
java.lang.Float
java.lang.Character
generics.GenericMethods
*///:~
```

**GenericMethods** 并不是参数化的，尽管这个类和其内部的方法可以被同时参数化，但是在这个例子中，只有方法 **f()** 拥有类型参数。
使用泛型类时，必须在创建对象的时候指定类型参数的值，而使用泛型方法的时候，通常不必指明参数类型，因为编译器会为我们找出具体的类型。这称为 <font face='kaiti'>类型参数推断</font>（type argument inference）。因此，我们可以像调用普通方法一样调用 **f()**。

擦除的神秘之处
--

尽管可以声明 **ArrayList.class**，但是不能声明 **ArrayList&lt;Integer&gt;.class**，

``` java
package generics;

import java.util.ArrayList;

public class ErasedTypeEquivalence {
    public static void main(String[] args) {
        Class string = new ArrayList<String>().getClass();
        Class integer = new ArrayList<Integer>().getClass();
        System.out.println(string == integer);
    }
} /*
true
*///:~
```
<font face='kaiti'>在泛型代码内部，无法获得任何有关泛型参数类型的信息。</font>

Java 泛型是使用擦除来实现的，这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，你唯一知道的就是你在使用一个对象。

只有当你希望使用的类型参数比某个具体类型（以及它的所有子类型）更加“泛化”时——也就是说，当你希望代码能够跨多个类工作时，使用泛型才有所帮助。

必须查看所有代码，并确定它是否“足够复杂”到必须使用泛型的程度。

> 擦除是 Java 为支持迁移兼容性、向后兼容性等作出的妥协。

擦除的补偿
--

擦除丢失了在泛型代码中执行某些操作的能力。任何在运行时需要知道确切类型信息的操作都将无法工作。

有时必须通过引入类型标签来对擦除进行补偿。这意味着你需要显式地传递你的类型的 Class 对象，以便你可以在类型表达式中使用它。

**instanceof **无法使用，因为类型信息已经被擦除了。如果引入类型标签，就可以使用动态的 **isInstance()**。

不能创建泛型数组，因为类型被擦除了。一般的解决方案是在任何想要创建泛型数组的地方都使用 **ArrayList**。

自限定的类型
---

``` java
package generics;

public class BasicHolder<T> {
    T element;
    void set(T arg) {
        element = arg;
    }
    T get() {
        return element;
    }
    void f() {
        System.out.println(element.getClass().getSimpleName());
    }

}

```

``` java
package generics;

class Subtype extends BasicHolder<Subtype> {

}
public class CRGWithBasicHolder {
    public static void main(String[] args) {
        Subtype st1 = new Subtype(), st2 = new Subtype();
        st1.set(st2);
        Subtype st3 = st1.get();
        st1.f();
    }
} /* Output:
Subtype
*///:~
```
