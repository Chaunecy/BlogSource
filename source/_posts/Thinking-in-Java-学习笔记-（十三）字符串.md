---
title: Thinking in Java - 学习笔记 - （十三）字符串
date: 2018-05-02 14:46:46
tags:
	- Thinking in Java
	- Java
	- 学习笔记
---

**可以证明，字符串是计算机程序设计中最常见的行为**。

不可变String
--

**String**对象是不可变的。**String**类中每一个看起来会修改**String**值的方法，实际上都是创建了一个全新的**String**对象，以包含修改后的字符串内容。

<!-- more -->

重载“+”与StringBuilder
--

不可变性会带来一定的效率问题。为**String**对象重载的“操作符”就是一个例子。

如果字符串操作比较简单，那就可以依赖编译器，它会自动生成一个StringBuilder来提高效率。来合理地构造最终的字符串结果。但是如果有循环操作，那么最好自己创建一个**StringBuilder**对象。

无意识的递归
--

如果你希望**toString()**方法打印出对象的地址，**不要**使用**this**关键字。

``` java
public class InfiniteRecursion {
    public String toString() {
        //这里发生了自动类型转换，由InfiniteRecursion类型（this）转换成String类型。
        //因为编译器看到“+”，而this不是String，于是会试着把this转换为String。
        //转换的方法正是调用this上的toString()方法，于是发生了递归调用
        return " InfiniteRecursion address: " + this + "\n";
    }
    public static void main(String[] args) {
        List<InfiniteRecursion> v = new ArrayList<>();
        for (int i = 0; i < 10; i++)
            v.add(new InfiniteRecursion());
        System.out.println(v);
    }
}
```

格式化输出
--

``` java
    System.out.printf("Row 1: [%-5d %4f]\n", 5, 5.2);
```

### 格式化说明符

```
    %[argument_index$][flags][width][.precision]conversion
```
**width**控制最小尺寸，默认右对齐，可以通过使用“-”来改变对齐方向。

**precision**指明最大尺寸。对浮点数是表示小数位数（默认6位），多则舍入，少则补零。无法用于整数。

正则表达式
--

[RegexGolf - 练习正则表达式的网站](https://alf.nu/RegexGolf#accesstoken=1lTOqnuyL8WGibyMlZ5j)
