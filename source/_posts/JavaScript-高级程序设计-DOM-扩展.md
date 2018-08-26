---
title: JavaScript 高级程序设计 - DOM 扩展
date: 2018-08-14 13:51:04
categories:
	- 学习笔记
tags:
	- 学习笔记
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## 选择符 API

众多　JavaScript 库中最常用的一项功能，就是根据 CSS 选择符选择与某个模式匹配的 DOM 元素。实际上，jQuery 的核心就是通过 CSS 选择符查询 DOM 文档取得元素的引用。

### querySelector() 方法

querySelector() 方法接收一个 CSS 选择符，返回与该模式匹配的第一个元素，如果没有找到匹配的元素，返回 null。

``` javascript
var myDiv = document.querySelector('#myDiv');

var selected = document.querySelector('.selected');

// 取得类为 'button' 的第一个元素
var img = document.body.querySelector('img.buttor');
```

<!-- more -->

通过 Document 类型调用 `querySelector()` 方法时，会在文档元素的范围内查找匹配的元素。而通过 `Element` 类型调用 `querySelector()` 方法时，只会在该元素后代元素的范围内查找匹配的元素。

###　querySelectorAll() 方法

与　`querySelector()` 方法类似，但返回所有匹配的元素。返回的是一个 NodeList 的实例。

具体来说，返回的值实际上是带有所有属性和方法的 NodeList，而其底层实现则类似于一组元素的快照，而非不断对文档进行搜索的动态查询。

### matchesSelector() 方法

如果调用元素与该选择符匹配，返回 true；否则，返回 false。

``` javascript
if (document.body.matchesSelector('body.page1')) {
	// true
}
```

## 元素遍历

Element Traversal API 为 DOM 元素添加了 5 个属性。

- childElementCount
- firstElementChild
- lastElementChild
- previousElementSibling
- nextElementSibling

``` javascript
var i, len, child = element.firstElementChild;

while(child != element.lastElementChild) {
	processChild(child);
	child = child.nextElementSibling;
}
```


## HTML5

### 插入标记

#### innerHtml 属性

在读模式下，innerHTML 属性返回与调用元素的所有子节点（包括元素、注释和文本节点）对应的 HTML 标记。在写模式下，innerHTML 会根据指定的值创建新的 DOM 树，然后用这个 DOM 树完全替换调用元素原先的所有大孩子节点。

``` html
<div id='content'>
	<p>This is a <strong>paragraph</strong> with a list following it.</p>
	<ul>
		<li>Item 1</li>
		<li>Item 2</li>
		<li>Item 3</li>		
	</ul>
</div>
```

对于上面的 `<div>` 元素来说，它的 innerHTML 属性会返回如下字符串。

``` html
	<p>This is a <strong>paragraph</strong> with a list following it.</p>
	<ul>
		<li>Item 1</li>
		<li>Item 2</li>
		<li>Item 3</li>		
	</ul>
```

在写模式下，innerHTML 的值会被解析为 DOM 子树，替换调用元素原来的所有子节点。因为它的值被认为是 HTML，所以其中的所有标签都会按照浏览器处理 HTML 的标准方式转换为元素（转换结果因浏览器而异）。如果设置的值仅是文本而没有 HTML 标签，那么结果就是设置纯文本。

#### outerHTML 属性

在读模式下，outerHTML 返回调用它的元素及所有子节点的 HTML 标签。在写模式下，outerHTML 会根据指定的 HTML 字符串创建新的 DOM 子树。然后用这个 DOM 子树完全替换调用元素。

#### insertAdjacentHTML() 方法

接收两个参数：插入位置和要插入的 HTML 文本。

#### 内存与性能问题

在使用 innerHTML、outerHTML 和 insertAdjacentHTML() 之前，最好手工删除要被替换的元素的所有事件处理程序和 JavaScript 对象属性。

### scrollIntoView() 方法

可以在所有 HTML 元素上调用。

当页面发生变化时，一般会用这个方法来吸引用户的注意力。实际上，为某个元素设置焦点也会导致浏览器滚动并显示出获得焦点的元素。