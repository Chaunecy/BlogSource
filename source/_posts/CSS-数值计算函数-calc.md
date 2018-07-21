---
title: CSS 数值计算函数 calc()
date: 2018-07-14 15:20:15
categories:
	- 编程语言
tags:
	- CSS
---

## calc() 使用

``` css
#id {
    width: 90%;
    width: -moz-calc(100% - 10px * 2);
    width: -webkit-calc(100% -10px * 2);
    width: calc(100% - 10px * 2);
}
```

浏览器支持情况可以在 [这里](https://www.w3schools.com/cssref/func_calc.asp) 查到。