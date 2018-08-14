---
title: JavaScript 高级程序设计 - DOM2 和 DOM3
date: 2018-08-14 13:50:07
categories:
	- 学习笔记
tags:
	- 学习笔记
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## 样式

### 访问元素的样式

任何支持 style 特性的 HTML 元素在 JavaScript 中都有一个对应的 style 属性。这个 style 对象是 CSSStyleDeclaration 的实例，包含着通过 HTML 的 style 特性指定的所有样式信息，但不包含与外部样式表或嵌入样式表经层叠而来的样式。

<!-- more -->

在 style 特性中指定的任何 CSS 属性都将表现为这个 style 对象的相应属性。对于使用短划线（background-image）的 CSS 属性名，必须将其转换成驼峰大小写形式，才能通过 JavaScript 来访问。

> float 属性不能直接转换，要写 cssFloat，IE 中则是 styleFloat。

## 遍历

“DOM2 级遍历和范围”模块定义了两个用于辅助完成顺序遍历 DOM 结构的类型：NodeIterator 和 TreeWalker。这两个类型能够基于给定的起点对 DOM 结构执行深度优先（depth-first）的遍历操作。