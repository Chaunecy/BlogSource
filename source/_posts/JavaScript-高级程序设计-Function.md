---
title: JavaScript 高级程序设计 - Function
date: 2018-07-29 15:38:10
categories:
	- 学习笔记
tags:
	- 学习笔记
	- JavaScrpit
	- Professional JavaScript for Web Developers
---

## Function 类型

每个函数都是 Function 类型的实例，而且都与其他引用类型一样具有属性和方法。

由于函数是对象，因此函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定。

## 函数声明

``` javascript
function sum(num1, num2) {
    return num1 + num2;
}

var sum = function (num1, num2) {
    return num1 + num2;
}

// 不推荐，两次解析代码，影响性能
var sum = new Function('num1', 'num2', 
                      'return num1 + num2');
```

<!-- more -->

## 没有重载

可以将函数名想象成指针。

``` javascript
var addSomeNumber = function (num) {
    return num + 100;
}

addSomeNumber = function (num) {
    return num + 200;
}

var result = addSomeNumber(100);
```

声明的第二个同名函数会覆盖前面的函数，所以 js 中没有函数重载。

> JavaScript 中的参数列表其实是一个 arguments 数组，实参数目与形参数目不一致也能正确解析。

## 函数声明与函数表达式

解析器会率先读取函数声明，并使其在执行任何代码之前可以访问；

至于函数表达式，则必须等到解析器执行到它所在的代码行，都会真正被解释执行。

除了什么时候可以通过变量访问函数这一点区别之外，函数声明与函数表达式的语法其实是等价的。

## 作为值的函数

``` javascript
function createComparisonFunction(propertyName) {
    return function(obj1, obj2) {
        var value1 = obj1[propertyName];
        var value2 = obj2[propertyName];
        
        if (value1 < value2) {
            return -1;
        } else if (value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    };
}
```

## 函数内部属性

在函数内部，有两个特殊的对象：arguments 和 this。

### arguments

其中 arguments 是一个类数组对象，包含着传入函数中的所有参数。虽然 arguments 的主要用途是保存函数参数，但这个对象还有一个名叫 callee 的属性，该属性是一个指针，指向拥有这个 arguments 对象的函数。

``` javascript
function factorial(num) {
    if (num <= 1) {
        return 1;
    } else {
        return num * arguments.callee(num - 1);
    }
}
```

使用 arguments.callee 可以避免紧密耦合现象。

``` javascript
var trueFactorial = factorial;

factorial = function() {
    return 0;
}

alert(trueFactorial(5));    // 120 
alert(factorial(5));        // 0
```

> 严格模式下运行时，访问 arguments.callee 会导致错误。

### this

this 引用的是函数据以执行的环境对象，当在网页的全局作用域中调用函数时，this 对象引用的就是 window。

``` javascript
window.color = 'red';
var o = { color: 'blue' };

function sayColor() {
    console.log(this.color);
}

sayColor();            // 'red'

o.sayColor = sayColor;
o.sayColor();          // 'blue'
```

## 函数属性和方法

每个函数都包含两个属性：length 和 prototype。

length 属性表示函数希望接收的命名参数的个数（形参个数）。

对于 ECMAScript 中的引用类型而言，prototype 是保存它们所有实例方法的真正所在。

每个函数都包含两个非继承而来的方法：apply() 和 call()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内 this 对象的值 。

apply() 方法接收两个参数：一个是在其中运行函数的作用域，另一个是参数数组。其中，第二个参数可以是 Array 的实例，也可以是 arguments 对象。

apply() 和 call() 真正强大的地方是能够扩充函数赖以运行的作用域。

``` javascript
window.color = 'red';
var o = {
    color: 'blue'
};

function sayColor() {
    console.log(this.color);
}

sayColor();			    // red

sayColor.call(this);	// red
sayColor.call(window);	// red
sayColor.call(o);		// blue
```

使用 call() 或 apply() 来扩充作用域的最大好处，就是对象不需要与方法有任何耦合关系。

ECMAScript 5 还定义了一种方法：bind()。这个 方法会创建一个函数实例，其 this 值 会被绑定到传给 bind() 函数的值。

``` javascript
window.color = 'red';
var o = {
    color: 'blue'
};

function sayColor() {
    console.log(this.color);
}
var objectSayColor = sayColor.bind(o);
objectSayColor();	// blue
```

