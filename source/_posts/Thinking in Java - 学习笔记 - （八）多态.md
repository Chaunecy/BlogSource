---
title: Thinking in Java - 学习笔记 - （八）多态
date: 2018-05-01 18:58:14
categories:
	- 学习笔记
tags: 
	- Thinking in Java
	- Java
---

**在面向对象的程序设计语言中，多态是继数据抽象和继承之后的第三种基本特征。**

多态通过分离<font face="kaiti">做什么</font>和<font face="kaiti">怎么做</font>，从另一角度将接口和实现分离开来。多态不但能够改善代码的组织结构和可读性，还能创建<font face="kaiti">可扩展</font>的程序——即无论在项目最初创建时还是在需要添加新功能时都可以“生长”的程序。


<!-- more -->


- “封装”通过合并特征和行为来创建新的数据类型。“
- 实现隐藏”则通过将细节“私有化”把接口和实现分离开来。
- 多态的作用则是消除类型之间的耦合关系。
- 继承允许将对象视为它自己本身的类型或其基类型来处理，而同一份代码也就可以毫无差别地运行在这些不同类型之上了。

> 多态也称作<font face="kaiti">动态绑定、后期绑定</font>或<font face="kaiti">运行时绑定</font>。


方法调用绑定
-------------------

将一个方法调用同一个方法主体关联起来被称作<font face="kaiti">绑定</font>。
若在程序执行前进行绑定（如果有的话，由编译器和连接程序实现），叫做<font face="kaiti">前期绑定</font>。

``` java
    public static void doSomething(Shape i) {
    // ...
        i.play();
    }
```
doSomething方法接受一个Shape引用，那么在这种情况下，编译器怎样才能知道这个Shape引用指向的是Circle对象，而不是Triangle对象或是Rectangle对象呢？实际上，编译器无法得知。

解决的办法就是<font face="kaiti">后期绑定</font>，它的含义就是在运行时根据对象的类型进行绑定。

Java中除了**static**方法和**final**方法（**private**方法属于**final**方法）之外，其他所有的方法都是后期绑定。

> **final**可以防止其他人覆盖该方法。但更重要的一点或许是：这样做可以有效地“关闭”动态绑定，或者说，告诉编译器不需要对其进行动态绑定。

### 缺陷：域与静态方法

一旦你了解了多态机制，可能就会开始认为所有事物都可以多态地发生。然而，只有普通的方法调用可以是多态的。例如，如果你直接访问某个域，这个访问就将在编译期进行解析。

``` java
class FieldAccess {
    public static void main(String[] args) {
        Parent parent = new Son();
        System.out.printf("parent.field = %s,\nparent.getField() = %s,\n", parent.field, parent.getField());
        Son son = new Son();
        System.out.printf("son.field = %s,\nson.getField() = %s\n", son.field, son.getField());
    }
}

class Parent {

    Parent() { }
    public String field = "Parent";
    public String getField() {
        return field;
    }

}

public class Son extends Parent {
    //    public static String s = "static field of Son";
    public Son(){
    }
    public String field = "Son";
    public String getField() {
        return field;
    }
} /*
输出结果
parent.field = Parent,    //注意这里
parent.getField() = Son,
son.field = Son,
son.getField() = Son
*/
```

如果某个方法是静态的，它的行为就不具有多态性。静态方法是与类，而并非与单个的对象相关联的。

构造器与多态
--

尽管构造器并不具有多态性（它们实际上是static方法，只不过该static声明是隐式的），但还是非常有必要理解构造器怎样通过多态在复杂的层次结构中动作。

### 构造器的调用顺序

一个例子：

``` java
class Counter {
    private static int i = 0;
    static int increment() {
        return i++;
    }
}
class Meal {
    static {
        System.out.printf("%2d. Meal(), static\n", Counter.increment());
    }
    {
        System.out.printf("%2d. Meal(), non-static\n", Counter.increment());
    }
    Meal() { System.out.printf("%2d. Meal()\n", Counter.increment()); }
}
class Bread {
    static {
        System.out.printf("%2d. Bread(), static\n", Counter.increment());
    }
    {
        System.out.printf("%2d. Bread(), non-static\n", Counter.increment());
    }
    Bread() { System.out.printf("%2d. Bread()\n", Counter.increment()); }
}
class Cheese {
    static {
        System.out.printf("%2d. Cheese(), static\n", Counter.increment());
    }
    {
        System.out.printf("%2d. Cheese(), non-static\n", Counter.increment());
    }
    Cheese() { System.out.printf("%2d. Cheese()\n", Counter.increment()); }
}
class Lunch extends Meal {
    static {
        System.out.printf("%2d. Lunch(), static\n", Counter.increment());

    }
    {
        System.out.printf("%2d. Lunch(), non-static\n", Counter.increment());
    }
    Lunch() { System.out.printf("%2d. Lunch()\n", Counter.increment()); }
}
class PortableLunch extends Lunch {
    static {
        System.out.printf("%2d. PortableLunch, static\n", Counter.increment());
    }

    PortableLunch() { System.out.printf("%2d. PortableLunch()\n", Counter.increment()); }
    {
        System.out.printf("%2d. PortableLunch(), non-static\n", Counter.increment());
    }
}


public class Sandwich extends PortableLunch {
    static {
        System.out.printf("%2d. Sandwich(), static\n", Counter.increment());
    }
    // 非static成员按顺序初始化
    private Bread b = new Bread();
    private Cheese c = new Cheese();

    public Sandwich() { System.out.printf("%2d. Sandwich()\n", Counter.increment()); }
    public static void main(String[] args) {
        new Sandwich();
    }
    {
        System.out.printf("%2d. Sandwich(), non-static\n", Counter.increment());
    }
}/* 输出结果
 0. Meal(), static
 1. Lunch(), static
 2. PortableLunch, static
 3. Sandwich(), static
 4. Meal(), non-static
 5. Meal()
 6. Lunch(), non-static
 7. Lunch()
 8. PortableLunch(), non-static
 9. PortableLunch()
10. Bread(), static
11. Bread(), non-static
12. Bread()
13. Cheese(), static
14. Cheese(), non-static
15. Cheese()
16. Sandwich(), non-static
17. Sandwich()
*/
```

**类初始化时构造器的调用顺序：**

1. 调用基类构造器

2. 按声明顺序调用成员的初始化方法

3. 调用导出类构造器的主体。


### 构造器内部的多态方法的行为

如果在一个构造器的内部调用正在构造的对象的某个动态绑定的方法，那会发生什么情况呢？

在一般的方法内部，动态绑定的调用是在运行时才决定的，因为对象无法知道它是属于方法所在的那个类，还是属于那个类的导出类。

``` java
class Glyph {
    void draw() {
        System.out.println("Glyph.draw()");
    }
    Glyph() {
        System.out.println("Glyph() before draw()");
        draw();
        System.out.println("Glyph() after draw()");
    }

}
class RoundGlyph extends Glyph {
    // 这里初始化为1，但是输出结果中却是0
    private int radius = 1;
    RoundGlyph(int r) {
        radius = r;
        System.out.println("RoundGlyph.draw(), radius = " + radius);
    }

    @Override
    void draw() {
        super.draw();
        System.out.println("RoundGlyph.draw(), radius = " + radius);
    }
}
public class PolyConstructors {
    public static void main(String[] args) {
        new RoundGlyph(5);
    }
}/* 输出结果
Glyph() before draw()
Glyph.draw()
RoundGlyph.draw(), radius = 0
Glyph() after draw()
RoundGlyph.draw(), radius = 5
*/
```


**初始化的实际过程是：**

1. 在其他任何事物发生之前，将分配给对象的存储空间初始化成二进制的零

2. 调用基类构造器。此时调用被覆盖后的draw()方法（要在调用RoundGlyph构造器<font face="kaiti">之前</font>调用，由于步骤1的缘故，我们此时会发现radius的值为0。

3. 按照声明的顺序调用成员的初始化方法。

4. 调用导出类的构造器主体。

这样做有一个优点，那就是所有东西都至少初始化成零（或者是某些特殊数据类型中与“零”等价的值），而不是仅仅留作垃圾。

我们应该对这个程序的结果相当震惊。在逻辑方面，我们做的已经十分完美，但它的行为却不可思议地错了，并且编译器也没有报错。

> 编写构造器时有一条有效的准则：“用尽可能简单的方法使对象进入正常状态；如果可以的话，避免调用其他方法”。

在构造器内唯一能够安全调用的那些方法是基类中的**final**方法（也适用于**private**方法，它们自动属于**final**方法）。这些方法不能被覆盖，因此也就不会出现上述令人惊讶的问题。


用继承进行设计
--

一条通用的准则是：“用继承表达行为间的差异，并用字段表达状态上的变化”。

### 向下转型与运行时类型识别

由于向上转型（在继承层次中向上移动）会丢失具体的类型信息，所以我们就想，通过向下转型——也就是在继承层次中向下移动——应该能够获取类型信息。
在Java中，所有转型都会得到检查！即使我们只是进行一次普通的另括弧形式的类型转换，在进入运行期时仍然会对其进行检查。这种在运行期间对类型进行检查的行为称作“运行时类型识别”（**RTTI**）。

