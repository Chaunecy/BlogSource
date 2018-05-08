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

### 分类：

1. 简单工厂模式（Simple Factory）

2. 工厂方法模式（Factory Method）

3. 抽象工厂模式（Abstract Factory）

### 简单工厂模式

组成：

1. 工厂类角色

2. 抽象产品角色

3. 具体产品角色

<!-- more -->

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

## 小结

何时使用工厂方法模式：

1. 当客户程序不需要知道要使用对象的创建过程。

2. 客户程序使用的对象存在变动的可能，或者根本就不知道使用哪一个具体的对象。

但简单工厂模式与工厂方法模式没有真正的代码的改动。在简单工厂模式中，新产品的加入在修改工厂角色中的判断语句；而在工厂方法模式中，要么将判断逻辑留在抽象工厂角色中，要么在客户程序中将具体工厂角色写死。而且产品对象创建条件的改变必然会引起工厂角色的修改。

面对这种情况，Java的反射机制与配置文件的巧妙结合突破了限制。

## 抽象工厂模式

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





