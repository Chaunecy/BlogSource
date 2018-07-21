---
title: CSS 变量
date: 2018-07-14 15:08:13
categories:
	- 编程语言
tags:
	- CSS
---

## 声明变量

变量名的前缀是两根连词线 `--`。变量名大小写敏感。

``` css 
#id {
    --foo: #000000;
    --bar: #ffffff;
}

:root {
    /*声明全局变量*/
}
```

<!-- more -->

## 使用变量

使用 `var()` 函数读取变量。

``` css
#id {
    color: var(--foo);
}
```

更多使用情况可以查看 [CSS 变量教程](http://www.ruanyifeng.com/blog/2017/05/css-variables.html)。