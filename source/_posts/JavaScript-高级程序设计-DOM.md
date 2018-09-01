---
title: JavaScript 高级程序设计 - DOM
date: 2018-08-14 13:49:27
categories:
	- 学习笔记
tags:
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## 节点层次

### Node 类型

每个 Node 都有 nodeName 和 nodeValue 两个属性。、

每个 Node 都有 childNodes 属性，其中保存着一个 NodeList 对象。NodeList 对象是基于 DOM 结构动态执行查询的结果，因此 DOM 结构能够自动反映在 NodeList 对象中。

每个节点都有一个 parentNode 属性，该属性指向文档树中的父节点。childNodes 列表中的所有节点都具有相同的父节点，使用列表中每个节点的 previousSibling 属性和 nextSibling 属性，可以访问同一列表中的其他节点。

父节点的 firstChild 和 lastChild 属性分别指向其 childNodes 列表中的第一个和最后一个节点。

所有节点都有 ownerDocument 属性，该属性指向表示整个文档的文档节点。

<!-- more -->

#### 操作节点

最常用的操作节点的方法是 appendChild();

如果传入到 appendChild() 中的节点已经是文档的一部分了，那结果就是将该节点从原来的位置转移到新位置。因此，如果在调用 appendCHild() 时传入了父节点的 firstChild，那么该节点就会成为父节点的最后一个子节点。

如果需要把节点放在 childNodes 列表中的某个特定位置，可以使用 insertBefore() 方法。这个方法接受两个参数：要插入的节点和作为参照的节点。

replaceChild() 接受的两个参数是：要插入的节点和要替换的节点。新插入节点的所有关系指针都会从被它替换的节点复制过来，从技术上讲，被替换的节点仍然还在文档中，但它在文档中已经没有了自己的位置。

被移除的节点将成为方法的返回值。

#### 其他方法

有两个方法是所有类型的节点都有的：cloneNode() 和 normalize()。

cloneNode() 方法接受一个布尔值参数，表示是否执行深复制。在参数为 true 的情况下，执行深复制，也就是复制节点及其整个子节点树；在参数为 false 的情况下，执行浅复制，即只复制节点本身。复制后返回的节点副本属于文档所有，但并没有为它指定父节点。除非通过 apendChild()，insertBefore() 或 replaceChild() 将它添加到文档中。

> cloneNode() 方法不会复制添加到 DOM 节点中的 	JavaScript 属性，例如事件处理程序等。 IE 在此存在 bug，即它会复制事件处理程序，所以建议在复制之前最好先移除事件处理程序。

### Document 类型

在浏览器中，document 对象是 HTMLDocument（继承自 Document 类型）的一个实例，表示整个 HTML 页面。而且，document 对象是 window 对象的一个属性，因此可以将其作为全局对象来访问。

#### 文档的子节点

DOM 标准规定 Document 节点的子节点可以是 DocumentType、 Element、rocessingInstructor 或 Comment，但还有两个内置的访问其子节点的快捷方式。第一个是 documentElement 属性，该属性始终指向 HTML 页面中的 &lt;html&gt; 元素。另一个是通过 childNodes 列表访问文档元素。

``` html
<html>
	<body>

	</body>
</html>
```

这个页面在经过浏览器解析后，其文档中只包含一个子节点，即 &lt;html&gt; 元素。可以通过 documentElement 或  childNodes 列表来访问这个元素，如下所示。

``` javascript
var html = document.documentElement;
console.log(html === document.childNodes[0]); // true
console.log(html === document.firstChild);    // true
```

作为 HTMLDocument 的实例，document 对象还有一个 body 属性，直接指向 &lt;body&gt; 元素。

Document 另一个可能的子节点是 DocumentType。通常将 &lt;!DOCTYPE&gt; 标签看成一个与文档其他部分不同的实体，可以通过 doctype 属性来访问它的信息。

``` javascript
var doctype = document.doctype;
```

浏览器对 document.doctype 的支持不一致，因此这个属性的用处很有限。

#### 文档信息

URL、domain 和 referrer 三个属性都与网页的请求有关。

URL 属性包含页面完整的 URL，domain 属性中只包含页面的域名，referrer 属性中保存着链接到当前页面的那个页面的 URL。

三个属性中，只有 domain 是可以设置的。但由于安全方面的限制，也并非可以给 domain 设置任何值。不能将属性设置为 URL 中不包含的域。

当页面中包含来自其他子域的框架或内嵌框架时，能够设置 document.domain 就非常方便了。由于跨域安全限制，来自不同子域的页面无法通过 JavaScript 通过。而通过将每个页面的 document.domain 设置为相同的值，这些页面就可以互相访问对方包含的 JavScript 对象。

如果域名一开始是“松散的”（loose），那么不能将它再设置为“紧绷的”（tight）.

``` javascript
// 假设页面来自于 p2p.wrox.com 域

document.domain = "wrox.com";     // 松散的（成功）

document.domain = "p2p.wrox.com"; // 紧绷的（失败）
```

#### 查找元素

取得特定的某个或某组元素的引用，然后再执行一些操作应该是最常见的 DOM 应用。

Document 类型为此提供了两个方法：getElementById() 和 getElementsByTagName()。

第三个方法是只有 HTMLDocument 类型才有的方法，是 getElementsByName()。

#### 特殊集合

除了属性和方法，document 对象还有一些特殊的集合。这些集合都是 HTMLCollection 对象，为访问文档常用的部分提供了快捷方式。
包括：

- document.anchors，包含文档中所有带 name 特性的 &lt;a&gt' 元素。 
- document.images，包含所有 &lt;img&gt; 元素。
- document.links
- document.forms

这个特殊集合始终可以通过 HTMLDocument 对象访问到，而且集合中的项也会随着当前文档内容的更新而更新。

#### DOM 一致性检测

``` javascript
var hasXmlDom = document.implementation.hasFeature('XML', '1.0');
```

#### 文档写入

将输出流写入到网页中，有四个方法：write()、writeln()、open()、close()

### Attr 类型

``` javascript
var attr = document.createAttribute('align');
attr.value = 'left';
element.setAttributeNode(attr);

console.log(element.attributes['align'].value); // 'left'
console.log(element.getAttribute['align']); // 'left'
```

## DOM 操作技术

### 动态样式

``` html
<link rel='stylesheet' type='text/css' href='style.css'>
```

使用 DOM 代码动态创建：

``` javascript
var link = document.createElement('link");
link.rel = 'stylesheet';
link.type = 'text/css';
link.href = 'style.css';
var head = document.getElementsByTagName('head')[0];
head.appendChild(link);  // 必须添加到 <head> 才能保证浏览器一致性
```

### 操作表格

通过 insertRow()，insertCell() 等方法来简化代码。

### 使用 NodeList

本质上说，所有 NodeList 对象都是在访问 DOM 文档时实时运行的查询。

``` javascript
var divs = document.getElementsByTagName('div'),
	i,
	div;

for (i = 0; i < divs.length; i++) {
	div = document.createElement('div');
	document.body.apendChild(div);
}
```

每次循环都要对条件 `i < divs.length` 求值，意味着会运行取得所有 `<div>` 元素的查询。考虑到循环体每次都会创建一个新 `<div>` 元素并将其添加到文档中，因此 `divs.length` 的值在每次循环后都会递增。

``` javascript
for (i = 0, len = divs.length; i < len; i++) {
	// ...
}
```
