---
title: JavaScript 高级程序设计 - 事件
date: 2018-08-14 13:53:42
categories:
	- 学习笔记
tags:
	- 学习笔记
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## 事件流

页面的哪一个部分会拥有某个特定的事件？

要明白这个问题问的是什么，可以想象画在一张纸上的一组同心圆。如果你把手指放在圆心上，那么你的手指指向的不是一个圆，而是纸上所有圆。在单击按钮的同时，你也单击了按钮的容器元素，甚至也单击了整个页面。

事件流描述的是从页面中接收事件的顺序。IE 的事件流是事件冒泡流，而 Netscape Communicator 的事件流是事件捕获流。

<!-- more -->

### 事件冒泡

IE 的事件流叫做事件冒泡（event bubbling），即事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。

### 事件捕获

Netscape Communicator 团队提出的另一种事件流叫做事件捕获（event capturing）。事件捕获的思想是不太具体的节点应该更早接收到事件，而最具体的节点就忘最后接收到事件。事件捕获的用意在于在事件到达预定目标之前捕获它。

建议使用事件冒泡，在有特殊需要时再使用事件捕获。

### DOM 事件流

“DOM2级事件” 规定的事件流包括三个阶段：事件捕获阶段、牌目标阶段和事件冒泡阶段。

多数支持 DOM 事件流的浏览器都实现了一种特定的行为：即使“DOM2级事件”规范明确要求捕获阶段不会涉及事件目标，但 IE9、Safari、Chrome 和 Opera 9.5 及更高版本都会在捕获阶段触发事件对象上的事件。结果，就是有两个机会在目标对象上面操作事件。

## 事件处理程序

事件就是用户或浏览器自身执行的某种动作。诸如 click、load 和 mouseover，都是事件的名字。而响应某个事件的函数就叫做事件处理程序。

###　HTML 事件处理程序

``` html
<input type="button" value="Click Me" onclick="try(showMessage();} catch(ex){}">
```

HTML 与 JavaScript 代码耦合太紧密。

### DOM0 级事件处理程序

通过 JavaScript 指定事件处理程序的传统方式，就是将一个函数赋值给一个事件处理程序属性。

``` javascript
var btn = document.getElementById('myBtn');
btn.onclick = function() {
	alert("Clicked");
}
```

使用 DOM0 级方法指定的事件处理程序被认为是元素的方法，因此，这时候的事件处理程序是在元素的作用域中运行；换句话说，程序中的 this 引用当前元素。

``` javascript
var btn = document.getElementById('myBtn');
btn.onclick = function() {
	console.log(this.id); // 'myBtn'
}
```

也可以删除通过 DOM0 级方法指定的事件处理程序，

``` javascript
btn.onclick = null;
```

### DOM2 级事件处理程序

“DOM2 级事件”定义了两个方法，用于处理指定和删除事件处理程序的操作：addEventListener() 和 removeEventListener()。所有 DOM 节点中都包含这两个方法，并且它们都接受 3 个参数：要处理的事件名，作为事件处理程序的函数和一个布尔值。最后这个布尔值参数如果是 true，表示在捕获阶段调用事件处理程序；如果是 false，表示在冒泡阶段调用事件处理程序。

``` javascript
var btn = document.getElementById('myBtn');
btn.addEventListener('click', function() {
	console.log(this.id);
}, false);
```

与 DOM0 级方法一样，这里添加的事件处理程序也是在其依附的作用域中运行。使用 DOM2 级方法添加事件处理程序的主要好处是可以添加多个事件处理程序。

通过 addEventListener() 添加的事件处理程序只能使用 removeEventListener() 来移除；移除时传入的参数与添加处理程序时使用的参数相同。这也意味着通过 addEventListener() 添加的匿名函数将无法移除。

``` javascrpit
var btn = document.getElementById('myBtn');
btn.addEventListener('click', function() {
	console.log(this.id);
}, false);

btn.removeEventListener('click', function() { // 没有用！
	console.log(this.id);
}, false);
```

传入 addEventListener() 与 removeEventListener() 的是完全不同的函数。重写的例子如下：

``` javascript
var btn = document.getElementById('myBtn');
var handler = function() {
	console.log(this.id);
}
btn.addEventListener('click', handler, false);

btn.removeEventListener('click', handler, false); // 有效！
```

## 事件对象

在触发 DOM 上的某个事件时，会产生一个事件对象 event，这个对象中包含着所有与事件有关的信息。包括导致事件的元素、事件的类型以及其他与特定事件相关的信息。例如，鼠标操作导致的事件对象中，会包含鼠标位置的信息，而键盘操作导致的事件对象中，会包含与按下的键有关的信息。

### DOM 中的事件对象

兼容 DOM 的浏览器会将一个 event 对象传入到事件处理程序中，无论指定事件处理程序时使用什么方法（DOM0 级或 DOM2 级），都会传入 event 对象。

``` javascript
var btn = document.getElementById('myBtn');
btn.addEventListener('click', function(event) {
	console.log(event.type);
}, false);
```

在事件处理程序内部，对象 this 始终等于 currentTarget 的值，机遇 target 则只包含事件的实际目标。

当点击下述例子中的按钮时：

``` javascript
document.body.onclick = function(event) {
	console.log(event.currentTarget === document.body);
	console.log(this === document.body);

	// 按钮是 click 事件真正的目标。
	console.log(event.target === document.getElementById('myBtn'));
}
```

要阻止特定事件的默认行为，可以使用 preventDefault() 方法，例如阻止链接导航的默认行为：

``` javascript
var link = document.getElementById('myLink');
link.onclick = function(event) {
	event.preventDefault();
};
```

只有 cancelable 属性设置为 true 的事件，才可以使用 perventDefault() 来取消其默认行为。

另外，stopPropagation() 方法用于立即停止事件在 DOM 层次中的传播，即取消进一步的事件捕获或冒泡。例如，直接添加到一个按钮的事件处理程序可以调用 stopPropagation()，从而避免触发注册在 document.body 上面的事件处理程序。

``` javascript
var btn = document.getElementById('myBtn');
btn.onclick = function(event) {
	console.log('Clicked');
	event.stopPropagation();
};

document.body.onclick = function(event) {
	console.log('Body clicked');
};
```

如果不调用 stopPropagation()，就会在单击按钮时出现两个警告框。

事件对象的 eventPhase 属性，可以用来确定事件当前正位于事件流的哪个阶段。

## 事件类型

“DOM3 级事件” 规定了以下几类事件：

- UI 事件，当用户与页面上的元素交互时触发
- 焦点事件
- 鼠标事件
- 滚轮事件
- 文本事件
- 键盘事件
- 合成事件，为 IME 输入字符时触发
- 变动事件，当底层 DOM 结构发生变化时触发

### UI 事件

现有的 UI 事件：

- DOMActivate
- load
- unload
- abort
- error
- select
- resize
- scroll

### 焦点事件

- blur
- DOMFocusIn
- DOMFocusOut
- focus
- focusin
- focusout

当焦点从页面中的一个元素移动到另一个元素，会依次触发下列事件：

1. focusout 在失去焦点的元素上触发；
2. focusin 在获取焦点的元素上触发；
3. blur 在失去焦点的元素上触发；
4. DOMFocusOut 在失去焦点的元素上触发；
5. focus 在获得焦点的元素上触发；
6. DOMFocusIn 在获得焦点的元素上触发。

### 鼠标与滚轮事件

9 个鼠标事件：

- click
- dbclick
- mousedown
- mouseenter
- mouseleave
- mousemove
- mouseout
- mouseover
- mouseup

#### 客户区坐标位置

鼠标事件都是在浏览器视口中的特定位置上发生的。这个位置信息保存在事件对象的 clientX 和 clientY 属性中。

#### 页面坐标位置

页面坐标通过事件对象的 pageX 和 pageY 属性，表示事件是在页面中的什么位置发生的。

在页面没有滚动的情况下，pageX(Y) 和 clientX(Y) 相等。

#### 屏幕坐标位置

通过 screenX 和 screenY 属性可以确定鼠标事件发生时鼠标指针相对于整个屏幕的坐标信息。

#### 修改键
DOM 规定了 4 个属性，表示修改键（Shift, Ctrl, Alt, Meta）的状态：shiftKey、ctrlKey、altKey 和 metaKey。这些属性都是布尔值，如果相应的键被按下了，则值为 true，否则值为 false。

#### 无障碍性问题

需要注意的问题

- 使用 click 事件执行代码
- 不要使用 onmouseover 向用户显示新的选项。
- 不要使用 dbclick 执行重要操作。键盘无法触发这个事件。

### 键盘与文本事件

有 3 个键盘事件：

- keydown
- keypress
- keyup

### 触摸与手势事件

触摸事件：

- touchstart
- touchmove
- touchend
- touch cancel

手势事件：

- gesturestart
- gesturechange
- gestureend

只有两个手指都触摸到事件的接收容器时都会触发这些事件。在一个元素上设置事件处理程序，意味着两个手指必须同时位于该元素的范围之内，才能触发手势事件。由于这些事件冒泡，所以将事件处理程序放在文档上也可以处理所有手势事件。

触摸事件和手势事件之间存在某种关系。当一个手指放在屏幕上时，会触发 touchstart 事件，如果另一个手指又放在了屏幕上，则会先触发 gesturestart 事件，随后触发基于该手指的 touchstart 事件。如果一个或两个手指在屏幕上滑动，将会触发 gesturechange 事件。但只要有一个手指移开，就会触发 gestureend 事件，紧接着又会触发基于该手指的 touchend 事件。

## 内存和性能

### 事件委托

对“事件处理程序过多”问题的解决方案就是事件委托。事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。
例如，click 事件会一直冒泡到 document 层次。也就是说，我们可以为整个页面指定一个 onclick 事件处理程序，而不必给每个可单击的元素分别添加事件处理程序。

``` javascript 
var target = event.target;

switch(target.id) {
	case 'doSomething':
		break;
	case 'sayHi':
		break;
}
```

### 移除事件处理程序

内存中留有那些过时不用的“空事件处理程序”（dangling event handler），也是造成 Web 应用程序内存与性能问题的主要原因。

在两种情况下，可能会造成上述问题。第一种情况就是从文档中移除带有事件处理程序的元素时。这可能是通过纯粹的 DOM 操作，例如使用 removeChild() 和 replaceChild() 方法，但更多地是发生在使用 innerHTML 替换页面中某一部分的时候。如果带有事件处理程序的元素被 innerHTML 删除了，那么原来添加到元素中的事件处理程序极有可能无法被当作垃圾回收。

``` html
<div id="myDiv">
	<input type="button" value="Click Me" id="myBtn">
</div>
<script>
	var btn = document.getElementById('myBtn');
	btn.onclick = function() {
		btn.onclick = null; // 移除事件处理程序
		document.getElementById('myDiv').innerHTML = 'Proccessing...';
	};
</script>
```

在事件处理程序中删除按钮也能阻止事件冒泡。目标元素在文档中是事件冒泡的前提。