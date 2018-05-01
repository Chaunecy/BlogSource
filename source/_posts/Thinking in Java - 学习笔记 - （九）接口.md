---
title: nine
date: 2018-05-01 19:03:46
tags:
	- Thinking in Java
	- Java
	- 学习笔记
---


**接口和内部类为我们提供了一种将接口与实现分离的更加结构化的方法。**

抽象类和抽象方法
--

Java提供一个叫做<font face="kaiti">抽象方法</font>（相当于C++的纯虚函数）的机制，这种方法是不完整的；仅有声明而没有方法体。

``` java
    abstract void f();
```

包含抽象方法的类叫做抽象类。如果一个类包含一个或多个抽象方法，该类必须被限定为抽象的。（否则，编译器就会报错）

如果从一个抽象类继承，并想创建该新类的对象，那么就必须为基类中的所有抽象方法提供方法定义。如果不这样做（可以选择不做），那么导出类便也是抽象类，且编译器将会强制我们用**abstract**关键字来限定这个类。

创建抽象类和抽象方法非常有用，因为它们可以使类的抽象性明确起来，并告诉用户和编译器打算怎样来使用它们。抽象类还是很有用的重构工具，因为它们使得我们可以很容易地将公共方法沿着继承层次结构向上移动。

接口
---

**interface**关键字产生一个完全抽象的类，它根本就没有提供任何具体实现。

一个接口表示：“所有<font face="kaiti">实现</font>了该特定接口的类看起来都像这样”。因此，任何使用某特定接口的代码都知道可以该接口的哪些方法，而且仅需知道这些。因此，接口被用来建立类与类之间的协议。

但是，**interface**不仅仅是一个极度抽象的类，因为它允许人们通过创建一个能够被身上转型为多种基类的类型，来实现某种类似多重继变种的特性。

可以选择在接口中显式地将方法声明为**public**的，但即使不这么做，它们也是**public**的。因此，当要实现一个接口时，在接口中被定义的方法必须被定义为是**public**的；否则，它们将只能得到默认的包访问权限，这样在方法被继承的过程中，其可访问权限就被降低了，这是Java编译器所**不允许**的。

完全解耦
---

``` java
import java.util.Arrays;

class Processor {
    public String name() {
        return getClass().getSimpleName();
    }

    Object process(Object input) {
        return input;
    }
}

class Upcase extends Processor {
    @Override
    String process(Object input) { // Covariant return
        return ((String) input).toUpperCase();
    }
}

class Downcase extends Processor {
    String process(Object input) {
        return ((String) input).toLowerCase();
    }
}

class Splitter extends  Processor {
    String process(Object input) {
        return Arrays.toString(((String) input).split(" "));
    }
}
public class Apply {
    public static void process(Processor p, Object s) {
        System.out.printf("Using Processor %s\n", p.name());
        System.out.println(p.process(s));
    }

    public static String s = "Disagreement with beliefs is by definition incorrect";
    public static void main(String[] args) {
        process(new Upcase(), s);
        process(new Downcase(), s);
        process(new Splitter(), s);
    }
} /* 输出结果
Using Processor Upcase
DISAGREEMENT WITH BELIEFS IS BY DEFINITION INCORRECT
Using Processor Downcase
disagreement with beliefs is by definition incorrect
Using Processor Splitter
[Disagreement, with, beliefs, is, by, definition, incorrect]
*/
```




Java中的多重继承
---

使用接口的核心原因：为了能够向上转型为多个基类型（以及由此而来的灵活性）
第二个原因：防止客户端程序员创建该类的对象，并确保这仅仅是建立一个接口。

通过继承来扩展接口
----

尽量避免函数同名。

适配接口
--

### 适配器模式
``` java
/*
 * 适配器模式
 */
class FilterAdaptor implements Processor {
    Filter filter;

    public FilterAdaptor(Filter filter) {
        this.filter = filter;
    }

    @Override
    public String name() {
        return filter.name();
    }

    @Override
    public Object process(Object input) {
        return filter.process((Waveform) input);
    }
}

public class FilterProcessor {
    public static void main(String[] args) {
        Waveform w = new Waveform();
        Apply.process(new FilterAdaptor(new LowPass(1.0)), new Waveform());
        Apply.process(new FilterAdaptor(new HighPass(2.0)), w);
        Apply.process(new FilterAdaptor(new BandPass(3.0, 4.0)), w);
    }
}

/*
 * 过滤器基类
 */
class Filter {
    public String name() {
        return getClass().getSimpleName();
    }

    public Waveform process(Waveform input) {
        return input;
    }
}

/*
 * 过滤器导出类
 */
class LowPass extends Filter {
    double cutoff;

    public LowPass(double cutoff) {
        this.cutoff = cutoff;
    }

    @Override
    public Waveform process(Waveform input) {
        return input;
    }
}

class HighPass extends Filter {
    double cutoff;

    public HighPass(double cutoff) {
        this.cutoff = cutoff;
    }

    public Waveform process(Waveform input) {
        return input;
    }
}

class BandPass extends Filter {
    double lowCutoff, highCutoff;

    public BandPass(double lowCut, double highCut) {
        lowCutoff = lowCut;
        highCutoff = highCut;
    }

    public Waveform process(Waveform input) {
        return input;
    }
}

/*
 * 处理对象
 */
class Waveform {
    private static long counter;
    private final long id = counter++;

    public String toString() {
        return "Waveform " + id;
    }
}
```

接口中的域
---

放入接口中的任何域自动是**static**和**final**的。

这些域不是接口的一部分，它们的值被存储在该接口的静态存储区域内。

嵌套接口
---

接口可以嵌套在类或其他接口中。

接口与工厂
---

接口是实现多重继承的途径，而生成遵循某个接口的对象的典型方式就是<font face="kaiti">工厂方法</font>设计模式。

```
package thinkinginjava;

public class Factories {
    public static void serviceConsumer(ServiceFactory fact) {
        Service s = fact.getService();
        s.method1();
        s.method2();
    }
    public static void main(String[] args) {
        serviceConsumer(new Implementation1Factory());
        serviceConsumer(new Implementation2Factory());
    }
}

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
}

class Implementation1Factory implements ServiceFactory {

    @Override
    public Service getService() {
        return new Implementation1();
    }
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
}
class Implementation2Factory implements ServiceFactory {

    @Override
    public Service getService() {
        return new Implementation2();
    }
} /* 输出结果
Implementation1: method1
Implementation1: method2
Implementation2: method1
Implementation2: method2
*/
```


总结
---

任何抽象性都应该是应真正的需求而产生的。当必需时应该重构接口而不是到处添加额外级别的间接性，并由此带来额外的复杂性。

恰当的原则应该是优先选择类而不是接口。从类开始，如果接口的必需性变得非常明确，那么就进行重构。接口是一种重要的工具，但是它们容易被滥用。

