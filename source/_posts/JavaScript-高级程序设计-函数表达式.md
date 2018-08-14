---
title: JavaScript 高级程序设计 - 函数表达式
date: 2018-08-11 13:05:05
categories:
	- 学习笔记
tags:
	- 学习笔记
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## 函数声明

函数声明的一个重要特征就是函数声明提升（function declaration hoisting），意思是在执行代码之前会先读取函数声明。这就意味着可能把函数声明放在调用它的语句后面。

## 递归

``` javascript
function factorial(num) {
    if (num < 1) {
        return 1;
    } else {
        return num * factorial(num - 1);
    }
}

var anotherFactorial = factorial;
factorial = null;
console.log(anotherFactorial(4)); // 会报错
```

使用 `arguments.callee` 来代替函数名，可以规避以上错误。但是在严格模式下无法访问这个属性。解决方法是使用命名函数表达式。

<!-- more -->

``` javascript
var myFunction = (function f(num) {
    if (num < 1) {
        return 1;
    } else {
        return num * f(num - 1);
    }
});
```

## 闭包

闭包是指有权访问另一个函数作用域中的变量的函数。

创建闭包的常见方式，就是在一个函数内部创建另一个函数。

> 作用域链
>
> 当某个函数第一次被调用时，会创建一个执行环境（execution context）及相应的作用域链，并把作用域链赋值给一个特殊的内部属性（即[[Scope]]）。然后，使用 `this.arguments` 和其他命名参数的值来初始化函数的活动对象（activation object）。

在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象添加到它的作用域链中。

闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。只在必要时使用闭包。

### 闭包与变量

作用域链的配置机制引出了一个值得注意的副作用，即闭包只能取得包含函数中任何变量的最后一个值。别忘了闭包所保存的是整个变量对象，而不是某个特殊的变量。

``` javascript
function createFunctions() {
    var result = [];
    for (var i = 0; i < 10; i++) {
        result[i] = function() {
            return i;
        }
    }
    return result;
}
```

这个函数会返回一个函数数组。表面上看，每个函数都应该返回自己的索引值。但实际上，每个函数都返回 10。因为每个函数的作用域链中都保存着 `createFunctions() ` 函数的活动对象，所以它们引用的老师同一个变量 `i`。当 `createFunctions()` 函数返回后，变量 `i` 的值是 10。

但是，我们可以通过创建另一个匿名函数强制让闭包的行为符合预期。

``` javascript
function createFunctions() {
    var result = [];
    for (var i = 0; i < 10; i++) {
        result[i] = (function (idx) {
            return function () {
                return idx;
            }
        }(i));
    }
    return result;
}
```

### 关于 this 对象

在闭包中使用 this 对象也可能导致一些问题。this 对象是在运行时基于函数的执行环境绑定的；在全局函数中，this 等于 window，而当函数被作为某个对象的方法调用时，this 等于那个对象。不过，匿名函数的执行环境具有全局性，因此其 this 对象通常指向 window。

``` javascript
var name = "The Window";

var object = {
    name: "My Object",
    getNameFunc: function() {
        // var that = this;
        return function() {
            return this.name;
            // return that.name;   // "The Object"
        }
    }
};

console.log(object.getNameFunc()());  // "The Window"
```

在几种特殊情况下，this 的值可能会意外地改变。

``` javascript
var name = "The Window";

var object = {
    name: "My Object",
    getName: function () {
        return this.name;
    }
};

console.log(object.getName());  // "The Object"
console.log((object.getName)());  // "The Object"
console.log((object.getName = object.getName)());  // "The Window"
```

### 内存泄漏

如果闭包的作用域链中保存着一个 HTML 元素，那么就意味着该元素将无法被销毁。

``` javascript

function assignHandler() {
    var element = document.getElementById("someElement");
    element.onclick = function() {
        console.log(element.id);
    }
}
```

由于匿名函数保存了一个对 `assignHandler()` 的活动对象的引用，因此应付导致无法减少 element 的引用数。

``` javascript
function assignHandler() {
    var element = document.getElementById("someElement");
    var id = element.id;  // 
    
    element.onclick = function () {
        console.log(id);  //
    } ;
    
    element = null;  // 解除对 DOM 对象的引用，确保正常回收内存
}
```

## 模仿块级作用域

JavaScript 中没有块级作用域的概念。对于多次声明同一个变量的情况，只会对后续的声明视而不见（但会执行后续声明中的变量初始化）。匿名函数可以用来模仿块级作用域并避免这个问题。

用作块级作用域（通常称为私有作用域）的匿名函数的语法如下所示。

``` javascript
(function() {
    // 这里是块级作用域
})();
```

> 使用 `()` 把函数声明转换为函数表达式。函数表达式后加 `()` 代表执行函数。

这种做法可以减少闭包占用的内存问题，因为没有指向匿名函数的引用。只要函数执行完毕，就可以立即销毁其作用域链了。

## 私有变量

严格来讲，JavaScript 中没有私有成员的概念；所有对象都是公有的。但是有一个私有变量的概念。任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数的外部访问这些变量。私有变量包括函数的参数、局部变量和在函数内部定义的其他函数。

``` javascript
function Person(name) {
    // 特权方法：有权访问私有变量和私有函数的公有方法
    this.getName = function() {
        return name;
    };
    this.setName = function(value) {
        name = value;
    };
}

var person = new Person("Nicholas");
console.log(person.getName());  // "Nicholas"
```

### 静态私有变量

通过在私有作用域中定义私有变量或函数，同样可以创建特权方法。

``` javascript
(function() {
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction() {
        return false;
    }
    
    // 初始化示经声明的变量，问题会创建一个全局变量。
    // 严格模式下会导致错误
    // 构造函数
    MyObject = function() {
        
    };
    
    // 公有/特权方法
    MyObject.prototype.publicMethod = function() {
        privateVariable++;
        return privateFunction();
    }
})();
```

以这种方式创建静态私有变量会因为使用原型而增进代码利用，但每个实例都没有自己的私有变量。到底是使用实例变量，还有静态私有变量，最终还是要视具体需求而定。

> 多查找作用域链中的一个层次，就会在一定程度上影响查找速度。

### 模块模式

模块模式（module pattern）是为单例创建私有变量和特权方法。

按照惯例，JavaScript 是以对象字面量的方式来创建单例对象的。

``` javascript
var singleton = {
    name: 'value',
    method: function() {
        // your code here
    }
};
```

模块模式通过为单例添加私有变量和特权方法能够使其得到增强，

``` javascript
var singleton = function() {
    
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction() {
        return false;
    }
    
    // 特权/公有方法和属性
    return {
        publicProperty: true,
        publicMethod: function() {
            privateVariable++;
            return privateFunction();
        }
    }
}();
```

从本质上讲，对象字面量定义的是单例的公共接口。这种模式在需要对单例进行某些初始化，同时又需要维护其私有变量时是非常有用的。

### 增强的模块模式

增强的模块模式适合那些单例必须是某种类型的实例，同时还必须添加某些属性和（或）方法对其加以增强的情况。

``` javascript
var singleton = function() {
    
    // 私有变量和私有函数
    var privateVariable = 10;
    function privateFunction() {
        return false;
    }
    
    var object = new CustomType();
    // 特权/公有方法和属性
	object.publicProperty = true;
    object.publicMethod = function() {
        privateVariable++;
        return privateFunction();
    }
    return object;
}();
```

## 小结

函数表达式的特点：

- 函数声明要求有名字，但函数表达式不需要。没有名字的函数表达式也叫做匿名函数。
- 递归函数应该始终使用 `arguments.callee` 来递归地调用自身，不要使用函数名——函数名可能会发生变化。

当在函数内部定义了其他函数时，就创建了闭包。闭包有权访问包含函数内部的所有变量，原理如下：

- 在后台执行环境中，闭包的作用域链包含着它自己的作用域、包含函数的作用域和全局作用域。
- 当函数返回了一个闭包时，这个函数的作用域将会一直在内存中保存到闭包不存在为止。

模仿块级作用域：

- 创建并立即调用一个函数，这样既可以执行其中的代码，又不会在内存中留下对该函数的引用。

闭包还可以用于在对象中创建私有变量。