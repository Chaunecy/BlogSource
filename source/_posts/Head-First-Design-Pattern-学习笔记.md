---
title: Head First Design Pattern - 学习笔记
date: 2018-05-04 11:21:01
categories:
	- 设计模式
tags:
	- 学习笔记
	- Head First Design Pattern
---

## 工厂模式

### 分类

1. 简单工厂模式（Simple Factory）
2. 工厂方法模式（Factory Method）
3. 抽象工厂模式（Abstract Factory）



<!-- more -->

### 简单工厂模式

组成：

1. 工厂类角色
2. 抽象产品角色
3. 具体产品角色


``` java
// 抽象产品角色
public interface Car {
	public void drive();
}

// 具体产品角色
public class Benz implements Car {
	public void drive() {
		System.out.println("Driving Benz");
	}
}

public class Bmw implements Car {
	public void drive() {
		System.out.println("Driving Bmw");
	}
}
public class Audi implements Car {
	public void drive() {
		System.out.println("Driving Audi");
	}
}

// 工厂类角色
public class Driver {
	// 工厂方法，注意 返回类型为抽象产品角色
	public static Car dirverCar(String s) throws Exception {
		if(s.equalsIgnoreCase("Benz"))
			return new Benz();
		else if(s.equalsIgnoreCase("Bmw"))
			return new Bmw();
		......
		else throw new Exception();
	}
}

// 欢迎暴发户登场
public class Magnate {
	public static void main(String[] args) {
		try {
			// 告诉司机今天坐奔驰
			Car car = Driver.driverCar("benz");
			// 下命令：开车
			car.drive();
		。。。
```

简单工厂模式对于产品部分符合开闭原则（对扩展开放；对修改封闭），但是工厂部分并不理想，对于工厂部分，工厂类成为了全能类。

解决方法是工厂方法模式。

### 工厂方法模式

工厂方法模式去掉了简单工厂模式中工厂方法的静态属性，使得它可以被子类继承。这样，在简单工厂模式里集中在工厂方法上的压力可以由工厂方法模式里不同的工厂子类来分担。

组成：

1. 抽象工厂角色：工厂方法模式的核心，与应用程序无关。是具体工厂角色必须实现的接口或者必须继承的父类。
2. 具体工厂角色： 含有和具体业务逻辑有关的代码。由应用程序调用心创建对应的具体产品的对象。
3. 抽象产品角色：
4. 具体产品角色

``` java
// 抽象产品角色，与简单工厂模式类似

// 抽象工厂角色
public interface Driver {
	public Car driverCar();
}

public class BenzDriver implements Driver {
	public Car driverCar() {
		return new Benz();
	}
}
public class BmwDriver implements Driver {
	public Car driverCar() {
		return new Bmw();
	}
}


public class Magnate {
	public static void main(String[] args) {
		try {
			Driver driver = new BenzDriver();
			Car car = driver.driverCar();
			car.drive();
		}
		......
	}
}
```

### 小结

何时使用工厂方法模式：

1. 当客户程序不需要知道要使用对象的创建过程。
2. 客户程序使用的对象存在变动的可能，或者根本就不知道使用哪一个具体的对象。

但简单工厂模式与工厂方法模式没有真正的代码的改动。在简单工厂模式中，新产品的加入在修改工厂角色中的判断语句；而在工厂方法模式中，要么将判断逻辑留在抽象工厂角色中，要么在客户程序中将具体工厂角色写死。而且产品对象创建条件的改变必然会引起工厂角色的修改。

面对这种情况，Java的反射机制与配置文件的巧妙结合突破了限制。

### 抽象工厂模式

奔驰车和宝马车是两个产品树（产品层次结构）；而奔驰跑车和宝马跑车就是一个产品族，他们都可以放到跑车家族中，因此功能有所关联。

抽象工厂模式和工厂方法模式的区别就在于需要创建对象的复杂程度上。

抽象工厂模式的用意为： 给客户端提供一个接口，可以创建多个产品族中的产品对象。

使用抽象工厂模式还要满足以下条件：

1. 系统中有多个产品族，而系统一次只可能消费其中一族产品。
2. 同属于同一个产品族的产品一起使用。

抽象工厂模式的各个角色与工厂方法的一样。

## 单例模式

### 定义与结构

单例模式又叫单态模式或者单件模式。在GOF书中给出的定义为：保证一个类仅有一个实例，并提供一个访问它的全局访问点。单例模式中的“单例”通常用来代表那些本质上具有唯一性的系统组件（或者路段资源）。比如文件系统等。

单例模式的目的就是要控制选定的类只产生一个对象，当然也允许在一定情况下灵活改变对象的个数。

如何实现单例模式：一个办法是将构造函数变为私有的（至少是受保护的），使得外面的类不能通过引用来产生对象；同时为了保证类的可能性，就必须提供一个自己的对象以及访问这个对象的静态方法。

单例模式可分为有状态的和无状态的。有状态的单例对象一般也是可变的单例对象，多个单态对象在一起就可以作为一个状态仓库一样向外提供服务。没有状态的单例对象也就是不变单例对象，仅用做提供工具函数。

### 实现

#### 饿汉式

``` java
public class Singleton {
	// 在自己内部定义自己一个实例
	// private，只供内部调用，
	// 在加载时实例化
	private static Singleton instance = new Singleton();
	// 将构造函数设置为私有
	private Singleton() {
	}
	// 静态工厂方法，提供了一个供外部访问得到对象的静态方法
	public static Singlton getInstance() {
		return instance;
	}
}
```

#### 懒汉式

``` java
public class Singleton {
	// 和上面有什么不同？
	private static Singleton instance = null;
	// 设置为私有的构造函数
	private Singleton() {
	}
	// 静态工厂方法，防止多线程环境中产生多个实例
	public static synchronized Singleton getInstance() {
		// 比上面有所改进，
		//将实例化延迟到第一次被引用的时候。
		if (instance == null)
			instance = new Singleton();
		return instance;
	}
}
```

以上两种实现方式均失去了多态性，不允许被继承。
GOF认为最好的一种方式是维护一张存有对象和对应名称的注册表（可以有**HashMap**来实现。

### 单例模式邪恶论

单例模式在Java中存在的陷阱

#### 多个虚拟机

当系统中的单例类被拷贝运行在多个虚拟机下的时候，在每一个虚拟机下都可以创建一个实例对象。

在使用EJB、JINI、RMI技术的分布式系统中，就忘避免使用存在状态的单例模式，因为一个有状态的单例类，在不同虚拟机上，各个单例对象保存的状态很可能是不一样的。

#### 多个类加载器

当存在多个类加载器加载类的时候，即使它们加载的是完全相同的类，也会被区别对待。因为不同的类加载器会使用不同的命名空间来区分同一个类。

在很多J2EE服务器上允许存在多个servlet引擎，而每个引擎是采用不同的类加载器的。

#### 错误的同步处理

#### 子类破坏了对象控制

类构造函数变得不再仅有，就有可能失去对对象的控制。

#### 串行化（可序列化）

为了使一个单例类变成可串行化的，仅仅在声明中实现**Serializable**是不够的。因为一个串行化的对象在每次反串行化的时候，都会创建一个新的对象，而不仅仅是一个对原有对象的引用。为了防止这种情况，可以在单例类中加入readResolve方法。（参考《Effective Java》第57条建议）

还有基于XML格式的对象串行化方式，也存在上述问题。


## 建造模式

### 定义与结构

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。可以理解为：将构造复杂对象的过程和组成对象的部件解耦。

建造模式的组成：

1. 抽象建造者角色
2. 具体建造者角色
3. 指导者角色
4. 产品角色

首先客户程序创建一个指导者对象，一个建造者角色，并将建造者角色传入指导者对象进行配置。然后，指导者按照步骤调用建造者的方法创建产品。最后客户程序从建造者或者指导者那里得到产品。

### 实现

``` java
import java.util.*;

import junit.framework.*;

//不同的媒体形式:
class Media extends ArrayList {
}

class Book extends Media {
}

class Magazine extends Media {
}

class WebSite extends Media {
}

// 进而包含不同的媒体组成元素:
class MediaItem {
    private String s;

    public MediaItem(String s) {
        this.s = s;
    }

    public String toString() {
        return s;
    }
}

class Chapter extends MediaItem {
    public Chapter(String s) {
        super(s);
    }
}

class Article extends MediaItem {
    public Article(String s) {
        super(s);
    }
}

class WebItem extends MediaItem {
    public WebItem(String s) {
        super(s);
    }

}

// 抽象建造者角色，它规范了所有媒体建造的步骤:
class MediaBuilder {
    public void buildBase() {
    }

    public void addMediaItem(MediaItem item) {
    }

    public Media getFinishedMedia() {
        return null;
    }

} 

//具体建造者角色
class BookBuilder extends MediaBuilder {
    private Book b;

    public void buildBase() {
        System.out.println("Building book framework");
        b = new Book();
    }

    public void addMediaItem(MediaItem chapter) {
        System.out.println("Adding chapter " + chapter);
        b.add(chapter);
    }

    public Media getFinishedMedia() {
        return b;
    }
}

class MagazineBuilder extends MediaBuilder {
    private Magazine m;

    public void buildBase() {
        System.out.println("Building magazine framework");
        m = new Magazine();
    }

    public void addMediaItem(MediaItem article) {
        System.out.println("Adding article " + article);
        m.add(article);
    }

    public Media getFinishedMedia() {
        return m;
    }
}

class WebSiteBuilder extends MediaBuilder {
    private WebSite w;

    public void buildBase() {
        System.out.println("Building web site framework");
        w = new WebSite();
    }

    public void addMediaItem(MediaItem webItem) {
        System.out.println("Adding web item " + webItem);
        w.add(webItem);
    }

    public Media getFinishedMedia() {
        return w;
    }
}

//指导者角色，也叫上下文
class MediaDirector {
    private MediaBuilder mb;

    public MediaDirector(MediaBuilder mb) {
        this.mb = mb; //具有策略模式相似特征的
    }

    public Media produceMedia(List input) {
        mb.buildBase();
        for (Iterator it = input.iterator(); it.hasNext(); ) mb.addMediaItem((MediaItem) it.next());
        return mb.getFinishedMedia();
    }
}

//测试程序——客户程序角色
public class BuildMedia extends TestCase {
    private List input = Arrays.asList(new MediaItem("item1"), new MediaItem("item2"), new MediaItem("item3"), new MediaItem("item4"));

    public void testBook() {
        MediaDirector buildBook = new MediaDirector(new BookBuilder());
        Media book = buildBook.produceMedia(input);
        String result = "book: " + book;
        System.out.println(result);
        assertEquals(result, "book: [item1, item2, item3, item4]");
    }

    public void testMagazine() {
        MediaDirector buildMagazine = new MediaDirector(new MagazineBuilder());
        Media magazine = buildMagazine.produceMedia(input);
        String result = "magazine: " + magazine;
        System.out.println(result);
        assertEquals(result, "magazine: [item1, item2, item3, item4]");
    }

    public void testWebSite() {
        MediaDirector buildWebSite = new MediaDirector(new WebSiteBuilder());
        Media webSite = buildWebSite.produceMedia(input);
        String result = "web site: " + webSite;
        System.out.println(result);
        assertEquals(result, "web site: [item1, item2, item3, item4]");
    }

    public static void main(String[] args) {
        junit.textui.TestRunner.run(BuildMedia.class);
    }
}
```


### 应用优点

建造模式的优点

- 建造模式可以使得产品内部的表象独立变化。
 > 在原来的工厂方法模式中，产品内部的表象是由产品自身来决定的；而在建造模式中则是“外部化”为由建造者来负责。这样定义一个新的具体建造者角色就可以改变产品的内部表象，符合“开闭原则”。
- 建造模式使得客户不需要知道大多产品的细节。
- 每一个具体建造者角色是毫无关系的。
- 建造模式可以对复杂产品的创建进行更加精细的控制。

### 扩展

区分建造模式与抽象工厂模式：

- 建造模式着重于逐步将组件装配成一个成品并向外提供成品，而抽象工厂模式着重于得到产品族中相关的多个产品对象
- 抽象工厂模式的应用是受限于产品族的，建造模式不会。


## 原型模式

### 定义与结构

原型模式属于对象创建模式，GOF给它的定义为：用原型实例指定创建对象的各类，并且通过拷贝这些原型创建新的对象。

原型模式的结构：

1. 客户角色：让一个原型克隆自己来得到一个新对象
2. 抽象原型角色：实现了自己的**clone**方法
3. 具体原型角色：被复制的对象。

### 分析

客户是怎么来使用这些角色的对象的呢？最简单的方式就是：

``` java
	先new一个具体原型角色作为样本
	Prototype p = new ConcretePrototype();
	......
	// 使用原型p克隆出一个新对象p1
	Prototype p1 = (Prototype)p.clone();
```

实际运用中，往往存在一个原型管理器

``` java
	// 使用原型管理器后，客户获得对象的方式
	Prototype p1 = PrototypeManager.getManager().getPrototype("ConcretePrototype");
```

原型管理器可以用**HashMap**实现，使用单例模式来实现控制。

``` java
class PrototypeManager {
    private static PrototypeManager pm;
    private Map prototypes = null;

    private PrototypeManager() {
        prototypes = new HashMap();
    }

    //使用单例模式来得到原型管理器的唯一实例
    public static PrototypeManager getManager() {
        if (pm == null) {
            pm = new PrototypeManager();
        }
        return pm;
    }

    public void register(String name, Object prototype) {
        prototypes.put(name, prototype);
    }

    public void unregister(String name) {
        prototypes.remove(name);
    }

    public Prototype getPrototype(String name) {
        if (prototypes.containsKey(name)) {
            //将清单中对应原型的复制品返回给客户
            return (Prototype) ((Prototype) prototypes.get(name)).clone();
        } else {
            Prototype object = null;
            try {
                object =(Prototype)Class.forName(name).newInstance();
                register(name, object);
            } catch (Exception e) {
                System.err.println("Class " + name + "没有定义!");
            }
            return object;
        }
    }
}
```

通过增加或者删除原型管理器中注册的对象，可以比其它创建型模式更方便地在运行时增加或者删除产品。


一个较为经典的例子：绩效考核软件要对今年的各种考核数据进行年度分析，而这一组数据是存放在数据库中的。一般我们会将这一组数据封装
在一个类中，然后将此类的一个实例作为参数传入分析算法中进行分析，得到的分析结果返回到类中相应的变量中。假设我们决定对这组数据还要做另外一种分析以对分析结果进行比较评定。这时对封装有这组数据的类进行**clone**要比再次连接数据库得到数据好的多。

### 总结

**clone**方法在Java实现中有着一定的弊端和风险，不建议使用。

## 适配器模式

### 定义和结构

将一个类的接口转换成客户希望的另一个接口。

适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

适配器模式的组成：

1. 目标（Target）角色：定义Client使用的接口。
2. 被适配（Adaptee）角色：这个角色有一个已存并使用了的接口，而这个接口是需要我们适配的。
3. 适配器（Adapter）角色：将被适配角色已有的接口转换为目标角色希望的接口。

``` java
class Circle extends Shape {
    //这里引用了 已存在的TextCircle
    private TextCircle tc;

    public Circle() {
        tc = new TextCircle(); //初始化
    }

    public void display() {
        tc.displayIt(); //在规定的方法里面调用 TextCircle 原来的方法
    }
}
```

**适配器模式与代理模式的主要区别：**

代理模式应用的情况是不改变接口命名的，而且是对已有接口功能的一种控制；而适配器模式则强调接口转换。



## 桥梁模式

### 定义与结构

将抽象部分与它的实现部分分离，使它们都可以独立地变化。

可以将抽象部分理解为“front-end”，实现部分理解为“back-end”。

系统设计中，总是充满了各种变数，采取什么样的方式可以较好的解决变化带给系统的影响？可以使用抽象类+子类实现的方法，但这可能造成子类数量爆炸。

当这颗继承树上一些子树存在了类似的行为，可以将这些行为提取出来，采用接口的方式提供，再以组合的方式将服务提供给原来的子类。从而达到前端和后端独立的变化，以及后端的重用。

桥梁模式的角色组成：

1. 抽象（Abstraction）角色：定义了抽象类的接口而且维护着一个指向实现角色的引用。
2. 精确抽象（RefinedAbstraction）角色：实现并扩充由抽象角色定义的接口。
3. 实现（Implementor）角色
4. 具体实现（ConcreteImplementor）角色


### 举例

应用实例：java.awt

一段教学代码

```
class Abstraction {
    //维护着一个指向实现（Implementor）角色的引用
    private Implementation implementation;

    public Abstraction(Implementation imp) {
        implementation = imp;
    }

    // 下面定义了前端（抽象部分）应该有的接口
    public void service1() {
		//使用了后端（实现部分）已有的接口
		//组合实现功能
        implementation.facility1();
        implementation.facility2();
    }

    public void service2() {
        implementation.facility2();
        implementation.facility3();
    }

    public void service3() {
        implementation.facility1();
        implementation.facility2();
        implementation.facility4();
    }

    // For use by subclasses:
    protected Implementation getImplementation() {
        return implementation;
    }
}

//抽象部分（前端）的精确抽象角色
class ClientService1 extends Abstraction {
    public ClientService1(Implementation imp) {
        super(imp);
    }

    //使用抽象角色提供的方法组合起来完成某项功能
	//这就是为什么叫精确抽象角色（修正抽象角色）
    public void serviceA() {
        service1();
        service2();
    }

    public void serviceB() {
        service3();
    }
}

//另一个精确抽象角色，和上面一样的被我省略了
class ClientService2 extends Abstraction {
    public ClientService2(Implementation imp) {
        super(imp);
    }
    // ......

    //这里是直接通过实现部分的方法来实现一定的功能
    public void serviceE() {
        getImplementation().facility3();
    }
}

//实现部分（后端）的实现角色
interface Implementation {
    //这个接口只是定义了一定的接口
    void facility1();

    void facility2();

    void facility3();

    void facility4();
}

//具体实现角色就是要将实现角色提供的接口实现
//并完成一定的功能
//这里省略了
class Implementation1 implements Implementation {
    @Override
    public void facility1() {

    }

    @Override
    public void facility2() {

    }

    @Override
    public void facility3() {

    }

    @Override
    public void facility4() {

    }
}
```

## 组合模式

### 定义与结构

将对象以树形结构组织起来，以达成“部分 - 整体”的层次结构，使得客户端对单个对象和组合对象的使用具有一致性。（想一想文件管理系统）

组合模式的组成：

1. 抽象构件（Component）角色
2. 树叶构件（Leaf）角色
3. 树枝构件（Composite）角色

### 安全性与透明性

在Component中声明所有的用来管理子类对象的方法，会带来一些安全性问题，因为树叶不存在子类。

在Composite中声明所有的用来管理子类对象的方法，由于叶子和分支有不同的接口，所以失去的透明性。

### 举例：JUnit


``` java
//Test 接口——抽象构件角色
public interface Test {
    /**
     * Counts the number of test cases that will be run by this test.
     */
    public abstract int countTestCases();

    /**
     * Runs a test and collects its result in a TestResult instance.
     */
    public abstract void run(TestResult result);
}

//TestSuite 类的部分有关源码——Composite 角色，它实现了接口 Test
public class TestSuite implements Test {
    //用了较老的 Vector 来保存添加的 test
    private Vector fTests = new Vector(10);
    private String fName;

    // ...

    /**
     * Adds a test to the suite.
     */
    public void addTest(Test test) {
        //注意这里的参数是 Test 类型的。这就意味着 TestCase 和 TestSuite 以及以后
        //实现 Test 接口的任何类都可以被添加进来
        fTests.addElement(test);
    }
    // ...

    /**
     * Counts the number of test cases that will be run by this test.
     */
    public int countTestCases() {
        int count = 0;
        for (Enumeration e = tests(); e.hasMoreElements(); ) {
            Test test = (Test) e.nextElement();
            count = count + test.countTestCases();
        }
        return count;
    }

    /**
     * Runs the tests and collects their result in a TestResult.
     */
    public void run(TestResult result) {
        for (Enumeration e = tests(); e.hasMoreElements(); ) {
            if (result.shouldStop())
                break;
            Test test = (Test) e.nextElement();
            //关键在这个方法上面
            runTest(test, result);
        }
    }
    //这个方法里面就是递归的调用了，至于你的 Test 到底是什么类型的只有在运行的时候得知

    public void runTest(Test test, TestResult result) {
        test.run(result);
    }
    // ...
}

//TestCase 的部分有关源码——Leaf 角色，你编写的测试类就是继承自它
public abstract class TestCase extends Assert implements Test {

    // ...

    /**
     * Counts the number of test cases executed by run(TestResult result).
     */
    public int countTestCases() {
        return 1;
    }

    /**
     * Runs the test case and collects the results in TestResult.
     */
    public void run(TestResult result) {
        result.run(this);
    }
    // ...
}
```


### 优缺点

优点：

1. 客户端调用简单。
2. 容易在组合体内加入对象部件，客户端不必因为加入了新的对象部件而更改代码。


## 装饰模式

### 定义与结构

也叫包装器模式（wrapper），动态地给一个对象添加一些额外的职责。

案例：系统某处需要一个能报警的Door，是要给现有的Door添加子类，不是把报警的方法添加给Door？

装饰模式的组成：

1. 抽象构件角色（Component）
2. 具体构件角色（Concrete Component）：被装饰者，定义一个将要被装饰增加功能的类。
3. 装饰角色（Decorator）：持有一个构件对象的实例，并定义了抽象构件定义的接口
4. 具体装饰角色（Concrete Decorator）

``` java
//教学代码，无法直接运行

public interface Test {
    /**
     * Counts the number of test cases that will be run by this test.
     */
    public abstract int countTestCases();

    /**
     * Runs a test and collects its result in a TestResult instance.
     */
    public abstract void run(TestResult result);
}

//具体构件对象，但是这里是个抽象类
public abstract class TestCase extends Assert implements Test {
    // ...

    public int countTestCases() {
        return 1;
    }
    //...

    public TestResult run() {
        TestResult result = createResult();
        run(result);
        return result;
    }

    public void run(TestResult result) {
        result.run(this);
    }
    // ....
}

//装饰角色
public class TestDecorator extends Assert implements Test {
    //这里按照上面的要求，保留了一个对构件对象的实例
    protected Test fTest;

    public TestDecorator(Test test) {
        fTest = test;
    }

    /**
     * The basic run behaviour.
     */
    public void basicRun(TestResult result) {
        fTest.run(result);
    }

    public int countTestCases() {
        return fTest.countTestCases();
    }

    public void run(TestResult result) {
        basicRun(result);
    }

    public String toString() {
        return fTest.toString();
    }

    public Test getTest() {
        return fTest;
    }
}

//具体装饰角色，这个类的增强作用就是可以设置测试类的执行次数
public class RepeatedTest extends TestDecorator {
    private int fTimesRepeat;

    public RepeatedTest(Test test, int repeat) {
        super(test);
        if (repeat < 0)
            throw new IllegalArgumentException("Repetition count must be > 0");
        fTimesRepeat = repeat;
    }

    //看看怎么装饰的吧
    public int countTestCases() {
        return super.countTestCases() * fTimesRepeat;
    }

    public void run(TestResult result) {
        for (int i = 0; i < fTimesRepeat; i++) {
            if (result.shouldStop())
                break;
            super.run(result);
        }
    }

    public String toString() {
        return super.toString() + "(repeated)";
    }
}
```

### 应用环境

1. 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责
2. 处理可以撤消的职责。
3. 当不能采用生成子类的方法进行扩充时。

### 透明和半透明

对于面向接口编程，应该尽量使客户程序不知道具体的类型，而应该对一个接口操作。

这就要求装饰角色和具体装饰角色要满足Liskov替换原则：

``` java
	Component c = new ConcreteComponent();
	Component c1 = new ConcreteDecorator(c);
```
这种方式被称为透明式。JUnit中就属于这种应用。

### 其它

采用 Decorator 模式进行系统设计往往会产生许多看上去类似的小对象，这些对象仅仅在他们相互连接的方式上有所不同，而不是它们的类或是它们的属性值有所不同。

尽管对于那些了解这些系统的人来说，很容易对它们进行定制，但是很难学习这些系统，排错也很困难。

这是 GOF 提到的装饰模式的缺点。

## 门面模式

### 定义与结构

门面模式（**facade**）又称外观模式。为子系统中的一组接口提供一个一致的界面。
![使用门面模式前后](Head-First-Design-Pattern-学习笔记\使用门面模式前后.PNG)

门面模式的组成：

1. 门面角色：被客户角色调用。
2. 子系统角色：没有任何**facade**角色的信息和链接
3. 客户角色：调用**facade**角色来完成要得到的功能。

### 举例

**Facade**模式的一个典型应用就是进行数据库连接。

我们可以把连接、查询等操作提取出来，封装到一个类中，这样只需要传适当的参数到类中就可以了。

### 使用环境和优点

使用环境：

1. 当你要为一个复杂子系统提供一个简单接口时。
2. 客户程序与抽象类的实现部分之间存在着很大的依赖性。
3. 需要构建一个层次结构的子系统时，使用facade模式定义子系统趾每层的入口点。

优点：

- 它对客户屏蔽子系统组件，因而减少了客户处理的对象的数目并使得子系统使用起来更加方便。
- 它实现了子系统与客户之间的松耦合关系，而子系统内部的功能组件往往是紧耦合的。松耦合关系使得子系统的组件变化不会影响到它的客户。  
 - **Facade** 模式有助于建立层次结构系统，也有助于对对象之间的依赖关系分层。
 - **Facade** 模式可以消除复杂的循环依赖关系。这一点在客户程序与子系统是分别实现的时候尤为重要。在大型软件系统中降低编译依赖性至关重要。在子系统类改变时，希望尽量减少重编译工作以节省时间。
 - 用**Facade** 可以降低编译依赖性，限制重要系统中较小的变化所需的重编译工作。
 - **Facade**模式同样也有利于简化系统在不同平台之间的移植过程，因为编译一个子系统一般不需要编译所有其他的子系统。
- 如果应用需要，它并不限制它们使用子系统类。因此你可以让客户程序在系统易用性和通用性之间加以选择


### 总结

门面模式对于使两层之间的调用粗颗粒化很有帮助，避免了大量细颗粒度的访问。

## 享元模式

### 引子

``` java
public class TestPattern {
	public static void main(String[] args) {
		String n = "Hello, World";
		String m = "Hello, World";
		System.out.println(n == m);
		m = m + "h";
		System.out.println(n == m);
	}
} /* output
true
false
*/
```

### 定义与分类

享元模式英文称为**"Flyweight Pattern"**。

定义：采用一个共享类来避免大量拥有相同内容的”小类“的开销。这种开销中最常见、最直观的影响就是增加了内存的损耗。

**如何共享**：享元模式区分了内蕴状态和外蕴状态，即共性和个性。

享元模式可以分为：单纯享元模式和复合享元模式。

### 结构

单纯享元模式的结构：

1. 抽象享元角色：为具体享元角色规定必须实现的方法，外蕴状态就是以参数的形式通过此方法传入。
2. 具体享元角色
3. 享元工厂角色：负责创建和管理享元
4. 客户端角色：维护对所有享元对象的引用，而且还需要存储对应的外蕴状态。


复合享元模式的结构：

1. 抽象字元角色：为具体享元角色规定必须实现的方法，外蕴状态就是以参数的形式通过此方法传入。
2. 具体享元角色
3. 复合享元角色：它所代表的对象是还可以共享的，并且可以分解成多个单纯享元对象的组合。
4. 享元工厂角色
5. 客户端角色

### 教学代码（没看懂）

``` java
//这便是使用了静态属性来达到共享
//它使用了数组来存放不同客户对象要求的属性值
//它相当于享元角色（抽象角色被省略了）
class ExternalizedData {
    static final int size = 5000000;
    static int[] id = new int[size];
    static int[] i = new int[size];
    static float[] f = new float[size];

    static {
        for (int i = 0; i < size; i++)
            id[i] = i;
    }
}

//这个类仅仅是为了给 ExternalizedData 的静态属性赋值、取值
//这个充当享元工厂角色
class FlyPoint {
    private FlyPoint() {
    }

    public static int getI(int obnum) {
        return ExternalizedData.i[obnum];
    }

    public static void setI(int obnum, int i) {
        ExternalizedData.i[obnum] = i;
    }

    public static float getF(int obnum) {
        return ExternalizedData.f[obnum];
    }

    public static void setF(int obnum, float f) {
        ExternalizedData.f[obnum] = f;
    }

    public static String str(int obnum) {
        return "id: " +
                ExternalizedData.id[obnum] +
                ", i = " +
                ExternalizedData.i[obnum] +
                ", f = " +
                ExternalizedData.f[obnum];
    }
}

//客户程序
public class FlyWeightObjects {
    public static void main(String[] args) {
        for (int i = 0; i < ExternalizedData.size; i++) {
            FlyPoint.setI(i, FlyPoint.getI(i) + 1);
            FlyPoint.setF(i, 47.0f);
        }
        System.out.println(
                FlyPoint.str(ExternalizedData.size - 1));
    }
} ///:~
```

### 使用优缺点

**优点**：大幅度降低内存中对象的数量
**缺点**：使得系统逻辑复杂化，且在一定程度上外蕴状态影响了系统的速度。

使用享元模式的条件：

1. 系统中有大量的对象，他们使系统的效率降低
2. 这些对象的状态可以分离出所需要的内外两部分。


## 代理模式

### 定义与结构

为其他对象提供一种代理以控制对这个对象的访问。

在一些情况下客户不想或者不能直接引用一个对象，而代理对象可以在客户和目标对象之间起到中介作用，去掉客户不能看到的内容和服务或者增添客户需要的额外服务。

**代理模式可以分为8种，以下为几种常见、重要的：**

1. 远程（Remote）代理：为一个位于不同的地址空间的对象提供一个局域代表对象。比如你可以将一个在世界某个角落的一台机器通过代理假想成你局域网中的一部分。
2. 虚拟（Virtual）代理：根据需要将一个资源消耗很大或者比较复杂的对象延迟到真正需要时才创建。比如加载一个很大的图片，用一个图片Proxy代替真正的图片
3. 保护（Protect or Access）代理：控制对一个对象的访问权限。比如论坛中身份不同权限不同。
4. 智能引用（Smart Reference）代理：提供比对目标对象额外的服务。比如记录访问的流量，提供一些友情提示等。



**代理模式的组成**

1. 抽象主题（Subject）角色：声明了真实主题和代理主题的共同接口
2. 代理主题角色：内部包含对真实主题的引用，并且提供和真实主题角色相同的接口。
3. 真实主题角色：定义真实的对象。

### 举例

以论坛中已注册用户和游客的权限不同来作为第一个例子。

**抽象主题角色**

``` java
public interface MyForum {
	public void AddFile();
}
```
**代理主题角色**

``` java
public class MyForumProxy implements MyForum {
    private RealMyForum forum = new RealMyForum();
    private int permission; //权限值

    public MyForumProxy(int permission) {
        this.permission = permission;
    }

    //实现的接口
    public void AddFile() {
        //满足权限设置的时候才能够执行操作
        //Constants 是一个常量类
        if (Constants.ASSOCIATOR == permission) {
            forum.AddFile();
        } else
            System.out.println("You are not a associator of MyForum ,please " +
                    "registe !");
    }
}
```

### 总结
代理模式能够协调调用者和被调用者，能够在一定程度上降低系统的耦合度。


## 责任链模式

### 定义与结构

定义：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

**适用范围**

1. 有多个对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定
2. 你想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求
3. 可处理一个请求的对象集合就被动态指定。

**角色组成**

1. 抽象处理者角色（Handler）：定义了一个处理请求的接口。也可以实现角色的后继链。
2. 具体处理者角色（Concrete Handler）：实现抽象角色中定义的接口，并处理它所负责的请求。如果不能处理则访问它的后继者。

### 纯与不纯

纯责任链模式：规定一个具体处理者角色只能对请求作出两种动作：自己处理；传给下家。

反之，就是不纯的责任链模式。



纯与不纯不重要，重要的是从中体味责任链模式的思想。

### 举例

代号自动生成器

``` java
//这是抽象处理者角色
public interface CodeAutoParse {
    //这里就是统一的处理请求使用的接口
    String[] generateCode(String moduleCode, int number, String rule, String[] target)
            throws BaseException;
}

//这个为处理日期使用的具体处理者
public class DateAutoParse implements CodeAutoParse {
    //获取当前时间
    private final Calendar currentDate = Calendar.getInstance();
    //这里用来注入下一个处理者，系统中采用 Spring Bean 管理
    private CodeAutoParse theNextParseOfDate;

    public void setTheNextParseOfDate(CodeAutoParse theNextParseOfDate) {
        this.theNextParseOfDate = theNextParseOfDate;
    }

    /*
     *实现的处理请求的接口
     *这个接口首先判断用户定义的格式是否有流水号,有则解析,没有则跳过
     *下传到下一个处理者
     */
    public String[] generateCode(String moduleCode, int number, String rule, String[] target)
            throws BaseException {
        //这里省略了处理的业务
        // ...
        if (theNextParseOfDate != null)
            return theNextParseOfDate.generateCode(moduleCode, number, rule,
                    target)
        else
            return target;
    }
}
```



### 其他

优点：降低耦合，提高灵活性。

但可能会带来一些额外的性能损耗，因为它每执行请求都要人开头开始遍历。



## 命令模式

### 定义与结构

**定义**：将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化；对请求排除或记录请求日志，以及支持可撤消的操作。

**组成**：

1. 命令角色（Command）：声明执行操作的接口。
2. 具体命令角色（Concrete Command）：将一个接收者对象绑定于一个动作；调用接收者相应的操作，以实现命令角色声明的执行操作的接口。
3. 客户角色（Client）：创建一个具体命令对象
4. 请求者角色（Invoker）：调用命令对象执行这个请求
5. 接收者角色（Receiver）：知道如何实施与执行一个请求相关的操作。任何类都可能作为一个接收者。

### 举例

``` java
// 命令角色————Action控制类
public class Action {
    // ...
    /*
     *可以看出，Action 中提供了两个版本的执行接口，而且实现了默认的空实现。
     */
    public ActionForward execute( ActionMapping mapping,
                                  ActionForm form,
                                  ServletRequest request,
                                  ServletResponse response)
            throws Exception {
        try {
            return execute(mapping, form, (HttpServletRequest) request,
                    (HttpServletResponse) response);
        } catch (ClassCastException e) {
            return null;
        }
    }
    public ActionForward execute( ActionMapping mapping,
                                  ActionForm form,
                                  HttpServletRequest request,
                                  HttpServletResponse response)
            throws Exception {
        return null;
    }
}
// 下面的就是请求者角色，它仅仅负责调用命令角色执行操作。
public class RequestProcessor {
    // ...
    protected ActionForward processActionPerform(HttpServletRequest request,
                                                 HttpServletResponse response,
                                                 Action action,
                                                 ActionForm form,
                                                 ActionMapping mapping)
            throws IOException, ServletException {
        try {
            return (action.execute(mapping, form, request, response));
        } catch (Exception e) {
            return (processException(request, response,e, form, mapping));
        }
    }
}
```

**Struts** 框架为我们提供了以上两个角色，要使用 **struts** 框架完成自己的业务逻辑，剩下
的三个角色就要由我们自己来实现了。步骤如下：

1. 很明显我们要先实现一个 **Action** 的子类，并重写 **execute()** 方法。在此方法中调用
    业务模块的相应对象来完成任务。
2. 实现处理业务的业务类，来充当接收者角色。
3. 配置**struts-config.xml**配置文件，将自己的**Action**和**Form**以及相应页面结合起来。
4. 编写 **jsp**，在页面中显式的制定对应的处理**Action**。


### Undo、事务及延伸

在定义中提到，命令模式支持可撤销的操作。

使用历史列表保存已经执行过的具体命令角色对象；修改具体命令角色中的执行方法，使它记录更多的执行细节，并将自己放入历史列表中；并在具体角色中添加**undo**方法，此方法根据记录的执行细节来复原状态。

同样，**redo**功能也能照此实现。

命令模式还有一个觉的用法就是执行事务操作。它可以在请求被传递到接收者角色之前，检验请求的正确性，甚至可以检查和数据库中数据的一致性，而且可以结合组合模式的结构，来一次执行多个命令。

### 优点及适用情况

命令模式有以下优点：

1. 命令模式将调用操作的请求对象与知道如何实现该操作的接收对象解耦。
2. 具体命令角色可以被不同的请求者角色重用。
3. 你可将多个命令装配成一个复合命令。
4. 增加新的具体命令角色很容易，因为这无需改变已有的类。

GOF 总结了命令模式的以下适用环境：

1. 需要抽象出待执行的动作，然后以参数的形式提供出来——类似于过程设计中的回调机
    制。而命令模式正是回调机制的一个面向对象的替代品。
2. 在不同的时刻指定、排列和执行请求。一个命令对象可以有与初始请求无关的生存期。
3. 需要支持取消操作。
4. 支持修改日志功能。这样当系统崩溃时，这些修改可以被重做一遍。
5. 需要支持事务操作。


### 总结

从面向对象的角度来看，命令模式是不完善的。命令角色仅仅包含一个方法，没有任何属性存在。这是将函数层面的人物提升到了类的层面。但命令模式很成功的解决了许多问题。



## 解释器模式

### 定义与结构

**定义**：定义语言的方法，并且建立一个解释器来解释该语言中的句子。

**组成**：

1. 抽象表达式角色：声明一个抽象的解释操作，这个接口为所有具体表达式角色（抽象语法树中的节点）都要实现的。
2. 终结符表达式角色：具体表达式。
   1. 实现与方法中的终结符相关联的解释操作
   2. 而且句子中的每个终结符需要该类的一个实例与之对应
3. 非终结符表达式角色：具体表达式
   1. 方法中的每条规则R::=R1R2...Rn都需要一个非终结符表达式角色
   2. 对于从R1到Rn的每个符号都维护一个抽象表达式角色的实例变量
   3. 实现解释操作，解释一般要递归地调用表示从R1到Rn的那些对象的解释操作
4. 上下文（环境）角色：包含解释器之外的一些全局信息。
5. 客户角色：
   1. 构建（或者被给定）表示该方法定义的语言中的一个特定的句子的抽象语法树
   2. 调用解释操作



### 举例

``` java
//上下文（环境）角色，使用 HashMap 来存储变量对应的数值
class Context {
    private Map valueMap = new HashMap();

    public void addValue(Variable x, int y) {
        Integer yi = new Integer(y);
        valueMap.put(x, yi);
    }

    public int LookupValue(Variable x) {
        int i = ((Integer) valueMap.get(x)).intValue();
        return i;
    }
}

//抽象表达式角色，也可以用接口来实现
abstract class Expression {
    public abstract int interpret(Context con);
}

//终结符表达式角色
class Constant extends Expression {
    private int i;

    public Constant(int i) {
        this.i = i;
    }

    public int interpret(Context con) {
        return i;
    }
}

class Variable extends Expression {
    public int interpret(Context con) {
        //this 为调用 interpret 方法的 Variable 对象
        return con.LookupValue(this);
    }
}

//非终结符表达式角色
class Add extends Expression {
    private Expression left, right;

    public Add(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    public int interpret(Context con) {
        return left.interpret(con) + right.interpret(con);
    }
}

class Subtract extends Expression {
    private Expression left, right;

    public Subtract(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    public int interpret(Context con) {
        return left.interpret(con) - right.interpret(con);
    }
}

class Multiply extends Expression {
    private Expression left, right;

    public Multiply(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    public int interpret(Context con) {
        return left.interpret(con) * right.interpret(con);
    }
}

class Division extends Expression {
    private Expression left, right;

    public Division(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    public int interpret(Context con) {
        try {
            return left.interpret(con) / right.interpret(con);
        } catch (ArithmeticException ae) {
            System.out.println("被除数为 0！");
            return -11111;
        }
    }
}

//测试程序，计算 (a*b)/(a-b+2)
public class Test {
    private static Expression ex;
    private static Context con;

    public static void main(String[] args) {
        con = new Context();
        //设置变量、常量
        Variable a = new Variable();
        Variable b = new Variable();
        Constant c = new Constant(2);
        //为变量赋值
        con.addValue(a, 5);
        con.addValue(b, 7);
        //运算，对句子的结构由我们自己来分析，构造
        ex = new Division(new Multiply(a, b), new Add(new Subtract(a, b), c));
        System.out.println("运算结果为：" + ex.interpret(con));
    }
}
```

### 优缺点

解释器模式提供了一个简单的方式来执行语法，而且容易修改或者扩展语法。

但是解释器对于复杂方法难以维护。



## 迭代器模式

### 定义与结构

迭代器（Iterator）模式，又叫游标（Cursor）模式。

**定义**：提供一种方法访问一个窗口对象中各个元素，而又不需暴露该对象的内部细节。

**组成**：

1. 迭代器角色（Iterator）：定义访问和遍历元素的接口。
2. 具体迭代器角色（Concrete Iterator）：实现迭代器接口，并要记录遍历中的当前位置
3. 容器角色（Container）：负责提供创建具体迭代器角色的接口
4. 具体容器角色（Concrete Container）：具体窗口角色实现创建具体迭代器角色的接口——这个具体迭代器角色与该窗口的结构相关。

注意，在迭代器模式中，具体迭代器角色和具体窗口角色是耦合在一起的——遍历算法是与容器的内部细节紧密相关的。为了使客户程序从与具体迭代器角色耦合的困境中脱离出来，避免具体迭代器角色的更换给客户程序带来的修改，迭代器模式抽象了具体迭代器角色，使得客户程序更具一般性和重用性。这被称为**多态迭代**。

### 举例

**迭代器模式的实现方式**：

1. 迭代器角色定义遍历的接口，但是没有规定由谁来控制迭代。
   1. 外部迭代器：客户程序来控制遍历的进程
   2. 内部迭代器：迭代器自身来控制迭代
2. 在迭代器模式中没有规定谁来实现遍历算法。

**Java Collection**中的迭代器的实现（现在有了泛型）：

``` java
// 迭代器角色，仅仅定义了遍历接口
public interface Iterator {
    boolean hasNext();
    Object next();
    void remove();
}
// 容器角色，这里以 List 为例。它也仅仅是一个接口，就不罗列出来了

// 具体容器角色，便是实现了 List 接口的 ArrayList 等类。为了突出重点这里指罗列和
// 迭代器相关的内容

// 具体迭代器角色，它是以内部类的形式出来的。AbstractList 是为了将各个具体容器
// 角色的公共部分提取出来而存在的。
public abstract class AbstractList extends AbstractCollection implements List {
    // ……
    //这个便是负责创建具体迭代器角色的工厂方法
    public Iterator iterator() {
        return new Itr();
    }

    //作为内部类的具体迭代器角色
    private class Itr implements Iterator {
        int cursor = 0;
        int lastRet = -1;
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public Object next() {
            checkForComodification();
            try {
                Object next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();
            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

### 实现自己的迭代器

**需要注意的问题**：

1. 要操作的容器有支持的接口
2. 在遍历的过程中，通过该迭代器进行容器元素的增减操作是否安全？
3. 在窗口中存在复合对象的情况，迭代器怎样才能支持深层遍历和多种遍历？

### 适用情况

**迭代器模式给容器的应用带来以下好处**：

1. 支持以不同的方式遍历一个容器角色。根据实现方式的不同，效果上会有差别。
2. 简化了容器的接口。但是在 java Collection 中为了提高可扩展性，容器还是提供了遍历的接口。
3. 对同一个容器对象，可以同时进行多个遍历。因为遍历状态是保存在每一个迭代器对象中的。

**迭代器模式的适用范围**：

1. 访问一个容器对象的内容而无需暴露它的内部表示。
2. 支持对容器对象的多种遍历。
3. 为遍历不同的容器结构提供一个统一的接口（多态迭代）。



## 调停者模式

### 定义与结构

**Mediator Pattern**，调停者模式，作者认为叫做“传递器模式”更为贴切一点。

**定义**：用一个调停对象来封装一系列的对象交互。

**组成**：

1. 抽象调停者（Mediator）角色：定义统一的接口用于各同事角色之间的通信
2. 具体调停者（Concrete Mediator）角色：具体调停者角色通过协调各同事角色实现协作行为。
3. 同事（Colleague）角色：每一个同事角色都知道对应的具体调停者角色，而且与其他的同事角色通信的时候，一定要通过调停者角色协作。

### 进一步讨论

**MVC**模型分为模型层（**Model**）、表现层（**View**）还有控制层（**Control**\\**Mediator**）。控制层便是位于表现层与模型层之间的调停者。

调停者模式在定义上比较松散，在结构上和观察者模式、命令模式十分相像；而应用目的又与门面模式有些相似。

门面模式是介于客户程序与子系统之间的，而调停者模式是介于子系统与子系统之间的。



### 总结

调停者模式的使用有着分层设计的雏形。分层是开发人员分离复杂系统时最常用、最普通的技术。

**J2EE**的三层结构估计大家都耳熟能详。而调停者模式的作用便是将关系错乱复杂、层次不清晰的对象群分割成两层或者三层。

调停者模式很容易在系统中应用，也很容易在系统中误用。

当系统出现了“多对多”交互复杂的对象群，不要急于使用调停者模式，而要先反思你的系统在设计上是不是合理。



## 备忘录模式

### 定义与结构

备忘录（Memento）模式又称标记（Token）模式。

**定义**：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将对象恢复到原先保存的状态。

**组成部分**

1. 备忘录（Memento）角色：存储“备忘发起角色”的内部状态。为了防止“备忘发起角色”以外的其他对象访问备忘录，备忘录实际上有两个接口，“备忘录管理者角色”只能看到备忘录提供的窄接口——对于备忘录角色中存放的属性是不可见的。“备忘发起角色”则只能看到一个宽接口——能够得到自己放入备忘录角色中的属性。
2. 备忘发起（Originator）角色：“备忘发起角色”创建一个备忘录，用心记录当前时刻它的内部状态。
3. 备忘录管理者（Caretaker）角色：负责保存好备忘录。不能对备忘录的内容进行操作或检查。

### 举例

在**Java**中可保存封装的方法

1. 采用两个不同的接口类来限制访问权限。

   实现比较简单，但规范约束往往没有力度

2. 采用内部类来控制访问权限

   ``` java
   class Originator {
       //这个是要保存的状态
       private int state = 90;
       //保持一个“备忘录管理者角色”的对象
       private Caretaker c = new Caretaker();

       //读取备忘录角色以恢复以前的状态
       public void setMemento() {
           Memento memento = (Memento) c.getMemento();
           state = memento.getState();
           System.out.println("the state is " + state + " now");
       }

       //创建一个备忘录角色，并将当前状态属性存入，托给“备忘录管理者角色”存放。
       public void createMemento() {
           c.saveMemento(new Memento(state));
       }

       //this is other business methods...
       //they maybe modify the attribute state
       public void modifyState4Test(int m) {
           state = m;
           System.out.println("the state is " + state + " now");
       }
       // 作为私有内部类的备忘录角色，它实现了窄接口，可以看到在第二种方法中宽接
       // 口已经不再需要

       //注意：里面的属性和方法都是私有的
       private class Memento implements MementoIF {
           private int state;

           private Memento(int state) {
               this.state = state;
           }

           private int getState() {
               return state;
           }
       }
   }

   //测试代码——客户程序
   public class TestInnerClass {
       public static void main(String[] args) {
           Originator o = new Originator();
           o.createMemento();
           o.modifyState4Test(80);
           o.setMemento();
       }
   }

   //窄接口
   interface MementoIF {
   }

   //“备忘录管理者角色”
   class Caretaker {
       private MementoIF m;

       public void saveMemento(MementoIF m) {
           this.m = m;
       }

       public MementoIF getMemento() {
           return m;
       }
   }
   ```

3. 使用**clone**方法来简化备忘录模式。不太推荐

### 适用情况

使用备忘录模式的前提

1. 必须保存一个对象在某一个时刻的（部分）状态，这样以后需要它时它才能恢复到先前的状态。
2. 如果用接口来让其他对象直接得到这些状态，将全暴露对象的实现细节并破坏对象的封装性。

## 观察者模式

小偷和放风的

### 定义与结构

观察者（Observer）模式又名发布-订阅（Publish/Subscribe）模式。

**定义**：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

> 单一职责原则：一个对象只做一件事情。

**组成部分**

1. 抽象目标角色（Subject）：目标角色知道它的观察者，可以胡任意多个观察者观察同一个目标。并且提供注册和删除观察者对象的接口
2. 抽象观察者角色（Observer ）：为那些在目标发生改变时需要获得通知的对象定义一个更新接口。
3. 具体目标角色（Concrete Subject）将有关状态存入各个具体观察者对象。当它的状态发生改变时，向它的各个观察者发出通知。
4. 具体观察者角色（Concrete Observer）：存储有关状态，这些状态应与目标的状态保持一致。

### 举例

**JUnit**

``` java
// 抽象观察者角色
public interface TestListener {
    /**
     * An error occurred.
     */
    public void addError(Test test, Throwable t);

    /**
     * A failure occurred.
     */
    public void addFailure(Test test, AssertionFailedError t);

    /**
     * A test ended.
     */
    public void endTest(Test test);

    /**
     * A test started.
     */
    public void startTest(Test test);
}
// 具体观察者角色，我们采用最简单的 TextUI 下的情况来说明（AWT，Swing 对于整
// 天做 Web 应用的人来说，已经很陌生了）

public class ResultPrinter implements TestListener {
    //省略好多啊，主要是显示代码
    //……
    //下面就是实现接口 TestListener 的四个方法
    //填充方法的行为很简单的说

    /**
     * @see junit.framework.TestListener#addError(Test, Throwable)
     */
    public void addError(Test test, Throwable t) {
        getWriter().print("E");
    }

    /**
     * @see junit.framework.TestListener#addFailure(Test, AssertionFailedError)
     */
    public void addFailure(Test test, AssertionFailedError t) {
        getWriter().print("F");
    }

    /**
     * @see junit.framework.TestListener#endTest(Test)
     */
    public void endTest(Test test) {
    }

    /**
     * @see junit.framework.TestListener#startTest(Test)
     */
    public void startTest(Test test) {
        getWriter().print(".");
        if (fColumn++ >= 40) {
            getWriter().println();
            fColumn = 0;
        }
    }
}

//下面只列出了当 Failures 发生时是怎么来通知观察者的
public class TestResult extends Object {
    //这个是用来存放测试 Failures 的集合
    protected Vector fFailures;
    //这个就是用来存放注册进来的观察者的集合
    protected Vector fListeners;

    public TestResult() {
        fFailures = new Vector();
        fListeners = new Vector();
    }

    /**
     * Adds a failure to the list of failures. The passed in exception
     * caused the failure.
     */
    public synchronized void addFailure(Test test, AssertionFailedError t) {
        fFailures.addElement(new TestFailure(test, t));
//下面就是通知各个观察者的 addFailure 方法
        for (Enumeration e = cloneListeners().elements(); e.hasMoreElements(); ) {
            ((TestListener) e.nextElement()).addFailure(test, t);
        }
    }

    /**
     * 注册一个观察者
     */
    public synchronized void addListener(TestListener listener) {
        fListeners.addElement(listener);
    }

    /**
     * 删除一个观察者
     */
    public synchronized void removeListener(TestListener listener) {
        fListeners.removeElement(listener);
    }

    /**
     * 返回一个观察者集合的拷贝，当然是为了防止对观察者集合的非法方式操作了
     * 可以看到所有使用观察者集合的地方都通过它
     */
    private synchronized Vector cloneListeners() {
        return (Vector) fListeners.clone();
    }
    // ……
}


public class TestRunner extends BaseTestRunner {
    private ResultPrinter fPrinter;

    public TestResult doRun(Test suite, boolean wait) {
        //就是在这里注册的
        result.addListener(fPrinter);
        //……
    }
}
```

### 使用情况

使用观察者模式的情况

1. 当一个抽象模型有两个方面，其中一个方面依赖于另一方面。
2. 当对一个对象的改变需要同时改变其它对象，而不知道有多少对象有待改变
3. 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。

### 我推你拉

观察者模式在关于目标角色、观察者角色通信的具体实现中，有两个版本。

一种情况便是目标角色在发生变化后，仅仅告诉观察者角色“我变化了”；观察者角色如果想要知道具体的变化细节，则就要自己从目标角色的接口中得到。这种模式被很形象的称为：拉模式——就是说变化的信息是观察者角色主动从目标角色中“拉”出来的。

还有一种方法，那就是我目标角色“服务一条龙”，通知你发生变化的同时，通过一个参数将变化的细节传递到观察者角色中去。这就是“推模式”——管你要不要，先给你。

这两种模式的使用，取决于系统设计时的需要。如果目标角色比较复杂，并且观察者角色进行更新时必须得到一些具体变化的信息，则“推模式”比较合适。如果目标角色比较简单，则“拉模式”就很合适了。

## 策略模式

### 定义与结构

策略（Strategy）模式属于对象行为型设计模式，主要是定义一系列的算法，把这些算法一个个封装成拥有共同接口的单独的类，并且使它们之间可以互换。

好处

- 可以将算法的使用和算法本身分离。
- 算法可以重用，也可以考虑全貌和享元模式来共享算法对象

**组成部分**

1. 算法使用环境（Context）角色：
2. 抽象策略（Strategy）角色：规定所有具体策略角色所需的接口。
3. 具体策略角色：实现接口

### 举例

java .awt 布局管理器

### 使用建议

- 系统需要能够在几种算法中快速的切换
- 系统中有一些类它们仅行为不同时
- 系统中存在多重条件选择语句时

策略模式中不可以同时使用多于一个算法。

## 状态模式

### 定义与结构

**定义**：允许一个对象在其内部状态改变时改变它的行为。

**组成部分**

1. 使用环境(Context)角色：定义客户程序需要的接口；维护一个具体状态角色的实例，这个实例决定当前的状态。
2. 状态（State）角色：定义一个接口以封装与使用环境角色的一个特定状态相关的行为。
3. 具体状态（Concrete State）角色：实现接口。

### 实现

状态模式与策略模式很像。

状态模式如何应对状态转换：

- 在每一个具体状态角色中来指定后续状态以及何时进行转换。
- 把状态转换规则写在.xml甚至是数据库中，使用Java的反射机制。

优势：

- 免除了复杂的逻辑判断语句
- 封装，使得增加新的状态更加简单

带来的问题：每个状态对应一个具体的状态类，使整体分散，逻辑不清晰

适用情况：

- 一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为
- 一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。

### 状态VS策略

两者的区别：区分这两个模式的关键是看行为是由状态驱动不是由一级算法驱动。

通常，State模式的“状态”是在对象内部的，Strategy模式的“策略”可以在对象外部。

## 模板模式

### 定义与结构

模板方法（Template Method）模式：定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。

**组成部分**

1. 抽象类（Abstarct Class）：定义了一到多个抽象方法；实现一个模板方法
2. 具体类（Concrete Class）：实现父类中的抽象方法以完成算法中与特定子类相关的步骤。

### 举例

``` java
    public void runBare() throws Throwable {
        setUp();
        try {
            runTest();
        }
        finally {
            tearDown();
        }
    }
// 未使用抽象方法，使用空的无为方法（钩子方法）
    protected void setUp() throws Exception {
    }
    protected void tearDown() throws Exception {
    }
```

### 适用情况

1. 一次性实现一个算法的不变的部分，并将可变的 行为留给子类实现
2. 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。
3. 控制子类扩展。

## 访问者模式

### 定义与结构

可以在不修改已有程序结构的前提下，通过添加额外的“访问者”来完成对已有代码的提升。

**定义**：表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

**组成结构**

1. 访问者角色（Vistor）：为该对象结构中具体元素角色声明一个访问操作接口。
2. 具体访问者角色（Concrete Visitor）：实现每个由访问者角色声明的操作。
3. 元素角色（Element）：定义一个Accept操作，它以一个访问者为参数。
4. 具体元素角色：实现Accept
5. 对象结构角色（Object Structure）：能枚举它的元素；可以提供一个高层的接口以允许访问者访问它的元素；可以是一个复合（组合模式）或是一个集合，如一个列表或一个无序集合。

### 举例

``` java
import java.util.*;
        import junit.framework.*;

//访问者角色
interface Visitor {
    void visit(Gladiolus g);

    void visit(Runuculus r);

    void visit(Chrysanthemum c);
}

// The Flower hierarchy cannot be changed:
//元素角色
interface Flower {
    void accept(Visitor v);
}

//以下三个具体元素角色
class Gladiolus implements Flower {
    public void accept(Visitor v) {
        /***************************************************/
        v.visit(this);
        /***************************************************/
    }
}

class Runuculus implements Flower {
    public void accept(Visitor v) {
        /***************************************************/
        v.visit(this);
        /***************************************************/
    }
}

class Chrysanthemum implements Flower {
    public void accept(Visitor v) {
        /***************************************************/
        v.visit(this);
        /***************************************************/
    }
}

// Add the ability to produce a string:
//实现的具体访问者角色
class StringVal implements Visitor {
    String s;

    public String toString() {
        return s;
    }

    public void visit(Gladiolus g) {
        s = "Gladiolus";
    }

    public void visit(Runuculus r) {
        s = "Runuculus";
    }

    public void visit(Chrysanthemum c) {
        s = "Chrysanthemum";
    }
}

// Add the ability to do "Bee" activities:
//另一个具体访问者角色
class Bee implements Visitor {
    public void visit(Gladiolus g) {
        System.out.println("Bee and Gladiolus");
    }

    public void visit(Runuculus r) {
        System.out.println("Bee and Runuculus");
    }

    public void visit(Chrysanthemum c) {
        System.out.println("Bee and Chrysanthemum");
    }
}

//这是一个对象生成器
//这不是一个完整的对象结构，这里仅仅是模拟对象结构中的元素
class FlowerGenerator {
    private static Random rand = new Random();

    public static Flower newFlower() {
        switch (rand.nextInt(3)) {
            default:
            case 0:
                return new Gladiolus();
            case 1:
                return new Runuculus();
            case 2:
                return new Chrysanthemum();
        }
    }
}

//客户测试程序
public class BeeAndFlowers extends TestCase {
    /*
    在这里你能看到访问者模式执行的流程：
    首先在客户端先获得一个具体的访问者角色
    遍历对象结构
    对每一个元素调用 accept 方法，将具体访问者角色传入
    这样就完成了整个过程
    */
    //对象结构角色在这里才组装上
    List flowers = new ArrayList();

    public BeeAndFlowers() {
        for (int i = 0; i < 10; i++)
            flowers.add(FlowerGenerator.newFlower());
    }

    Visitor sval;

    public void test() {
        // It's almost as if I had a function to
        // produce a Flower string representation:
        //这个地方你可以修改以便使用另外一个具体访问者角色
        sval = new StringVal();
        Iterator it = flowers.iterator();
        while (it.hasNext()) {
            /***************************************************/
            ((Flower) it.next()).accept(sval);
            /***************************************************/
            System.out.println(sval);
        }
    }

    public static void main(String args[]) {
        junit.textui.TestRunner.run(BeeAndFlowers.class);
    }
}
```

### 双重分派

首先在客户程序中将具体访问者模式作为参数传递给具体元素角色（加标记的地方所示）。这便完成了一次分派。

进入具体元素角色后，具体元素角色调用作为参数的具体访问者模式中的**visitor**方法，同时将自己（**this**）作为参数传递进去。具体访问者模式再根据参数的不同来选择方法来执行（加标记的地方所示）。这便完成了第二次分派。

### 优缺点及适用情况

如果一个操作的需要有变，那么仅仅修改一个具体访问者角色，而不用改动整个类层次。

如果系统中的类层次发生了变化，会对访问者模式产生根本影响——修改访问者角色和具体访问者角色。

《设计模式》一书中给出了访问者模式适用的情况：

1. 一个对象结构包含很多类对象，它们有不同的接口，而你想对这些对象实施一些依赖于其具体类的操作。
2. 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而你想避免让这些操作“污染”这些对象的类。Visitor 使得你可以将相关的操作集中起来定义在一个类中。
3. 当该对象结构被很多应用共享时，用 Visitor 模式让每个应用仅包含需要用到的操作。
4. 定义对象结构的类很少改变，但经常需要在此结构上定义新的操作。改变对象结构类需要重定义对所有访问者的接口，这可能需要很大的代价。如果对象结构类经常改变，那么可能还是在这些类中定义这些操作较好。


### 总结

这是一个巧妙而且复杂的模式，它的使用条件比较苛刻。当系统中存在着固定的数据结构（比如上面的类层次），而有着不同的行为，那么访问者模式也许是个不错的选择。

## 分派

### 名词解释

在 OO（object-oriented）语言中使用了继承来描述不同的类之间的“社会关系”——类型层次。而这些类实例化的对象们则是对这个类型层次的体现。因此大部分 OO 语言的对象都存在两个身份证：静态类型和实际类型。所谓静态类型，就是对象被声明时的类型；而实际类型则便是创建对象时的类型。

举个例子：B 是 A 的子类。则

```java
A object1 = new B ( );
```

中 object1 的静态类型便是 A，而实际类型却是 B。在 Java 语言中，编译器会根据对象的静态类型来检查错误；而在运行时，则使用对象的真实身份。

OO 还有一个重要的特点：一个类中可以存在两个相同名称的方法，而它们是根据参数类型的不同来区分的。正因以上两个原因，便产生了分派——根据类型的不同来选择不同的方法的过程——OO语言的重要特性。

### 分类

分派可以发生在编译期或者是运行期。因此按此标准，分派分为静态分派和动态分派。

在程序的编译期，只有对象的静态类型是有效的，因此静态分派就是根据对象（包括参数对象）的静态类型来选择方法的。最典型的便是方法重载（overloading）。

在运行期，动态分派会根据对象的实际类型来选择方法。典型的例子便是方法重置（overriding）。

而 OO 语言正是由以上两种分派方式来提供多态特性的。

按照选择方法时所参照的类型的个数，分派分为单分派和多分派。OO 语言也因此分为了单分派（Uni-dispatch）语言和多分派（Multi-dispatch）语言。比如 Smalltalk 就是单分派语言，而 CLOS 和 Cecil 就是多分派语言。

说道多分派，就不得提到另一个概念：多重分派（multiple dispatch）。它指由多个单分派组成的分派过程（而多分派则往往不能分割的）。因此单分派语言可以通过多重分派的方式来实现和多分派语言一样的效果。

那么我们熟悉的 Java 语言属于哪一种分派呢？

### Java分派实践

#### Override

``` java
public class Test {
    public static void main(String[] rags) {
        Father f = new Father();
        Father s = new Son();
        f.dost(1);
        s.dost(4);
        s.dost("dispatchTest");
	    //s.dost("test" , 5);
    }
}

class Father {
    public void dost(int i) {
        System.out.println("Welcome Father! int = " + i);
    }

    public void dost(String s) {
        System.out.println("Welcome Father! String = " + s);
    }
}

class Son extends Father {
    public void dost(int i) {
        System.out.println("Welcome Son! int = " + i);
    }

    public void dost(String s, int i) {
        System.out.println("Welcome Son! String = " + s + " int = " + i);
    }
}
```

在编译期间，编译器根据**f**、**s**的静态类型来为他们选择了方法，当然都选择了父类**Father**的方法。而到了运行期，则又根据**s**的实际类型动态的替换了原来选择的父类中的方法。这便是结果产生的原因。

如果把上面代码中的注释去掉，则会出现编译错误。原因便是在编译期，编译器根据**s**的静态类型**Father**找不到带有两个参数的方法**dost**。

#### Overload

``` java
public class Test {
    //这几个方法，唯独的不同便在这参数上
    public void dost(Father f, Father f1) {
        System.out.println("ff");
    }

    public void dost(Father f, Son s) {
        System.out.println("fs");
    }

    public void dost(Son s, Son s2) {
        System.out.println("ss");
    }

    public void dost(Son s, Father f) {
        System.out.println("sf");
    }

    public static void main(String[] rags) {
        Father f = new Father();
        Father s = new Son();
        Test t = new Test();
        t.dost(f, new Father());
        t.dost(f, s);
        t.dost(s, f);
    }
}

class Father {
}

class Son extends Father {
}
```

执行结果没有像预期的那样输出**ff**、**fs**、**sf**而是输出了三个**ff**。为什么？原因便是在编译期，编译器使用 s 的静态类型为其选择方法，于是这三个调用都选择了第一个方法；而在运行期，由于**Java**仅仅根据方法所属对象的实际类型来分派方法，因此这个“错误”就没有被纠正而一直错了下去……

可以看出，**Java**在静态分派时，可以根据**n**（n>0）个参数类型来选择不同的方法，这按照上面的定义应该属于多分派的范围。而在运行期时，则只能根据方法所属对象的实际类型来进行方法的选择，这又属于单分派的范围。

因此可以说**Java**语言支持静态多分派和动态单分派。

### 小插曲

``` java
public class Test {
    public static void main(String[] rags) {
        Father f = new Father();
        Father s = new Son();
        System.out.println("f.i " + f.i);
        System.out.println("s.i " + s.i);
        f.dost();
        s.dost();
    }
}

class Father {
    int i = 0;

    public void dost() {
        System.out.println("Welcome Father!");
    }
}

class Son extends Father {
    int i = 9;

    public void dost() {
        System.out.println("Welcome Son!");
    }
}
/* 运行结果
f.i 0
s.i 0
Welcome Father!
Welcome Son!
*/
```

产生的原因是**Java**编译和运行程序的机制。“数据是什么”是由编译时决定的；而“方法是哪个”则在运行时决定。

### 双重分派

Java 不能支持动态多分派，但是可以通过代码设计来实现动态的多重分派。这里举一个双重分派的实现例子。

大致的思想便是通过一个参数来传递**JVM**不能判断的类型。通过**Java**的动态单分派来完成一次分派后，在方法中使用**instanceof**来判断参数的类型，进而决定执行哪个相关方法。

``` java
public class Test {
    public static void main(String[] rags) {
        Father f = new Father();
        Father s = new Son();
        s.dosh(f);
        s.dosh(s);
        f.dosh(s);
        f.dosh(f);
    }
}

class Father {
    public void dosh(Father f) {
        if (f instanceof Son) {
            System.out.println("Here is Father's Son");
        } else if (f instanceof Father) {
            System.out.println("Here is Father's Father");
        }
    }
}

class Son extends Father {
    public void dosh(Father f) {
        if (f instanceof Son) {
            System.out.println("Here is Son's Son");
        } else if (f instanceof Father) {
            System.out.println("Here is Son's Father");
        }
    }
}/* 执行结果：
Here is Son's Father
Here is Son's Son
Here is Father's Son
Here is Father's Father
*/
```

用这种方式来实现双重分派，思路比较简单清晰。但是对于复杂一点的程序，则代码显得冗长，不易读懂。而且添加新的类型比较麻烦，不是一种好的设计方案。

访问者（Visitor）模式则较好的解决了这种模式的不足。