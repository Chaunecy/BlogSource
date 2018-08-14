---
title: JavaScript 高级程序设计 - BOM
date: 2018-08-11 15:38:42
categories:
	- 学习笔记
tags:
	- 学习笔记
	- 编程语言
	- JavaScript
	- Professional JavaScript for Web Developers
---

## window 对象

BOM 的核心对象是 window，它表示浏览器的一个实例。在浏览器中，window 对象有双重角色，它既是通过 JavaScript 访问浏览器窗口的一个接口，又是 ECMAScript 规定的 Global 对象，因此有权访问 `parseInt()` 等方法。

<!-- more -->

### 全局作用域

全局变量会成为 window 对象的属性。但定义全局变量与在 window 对象上直接定义属性还是有一点差别：全局变量不能通过 delete 操作符删除，而直接在 window 对象上的定义的属性可以。

使用 `var` 语句添加的 window 属性有一个名为 `[[Configurable]]` 的特性，这个 特性的值被设置为 false，因此这样定义的属性不可以通过 delete 操作符删除。

另外，尝试访问未声明的变量会抛出错误，但是通过查询 window 对象，可以知道某个可能未声明的变量是否存在。

### 窗口关系及框架

### 窗口位置

### 窗口大小

### 导航和打开窗口

### 间歇调用和超时调用

JavaScript 是单线程语言，但它允许通过设置超时值和间歇值来调度代码在特定的时刻执行。前者是在指定的时间过后执行代码，而后者则是每隔指定的时间就执行一次代码。

> 使用超时调用来模拟间歇调用是一种最佳模式。因为后一个间歇调用可能会在前一个间歇调用结束之前启动。

### 系统对话框

## location 对象

## navigator 对象

### 检测插件

### 注册处理程序

## sceern 对象

## history 对象

