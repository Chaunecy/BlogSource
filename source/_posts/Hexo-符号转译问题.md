---
title: Hexo 符号转译问题
date: 2018-07-02 11:36:21
categories:
	- 问题记录
tags:
	- Hexo
---

## Template render error

​当代码块以外的位置出现 &#123;&#123; &#125;&#125;  时，会抛出 template render error。

这里需要对符号进行转译。

以下为常见转译：

<!-- more -->

``` 
! &#33; — 惊叹号 Exclamation mark
” &#34; &quot; 双引号 Quotation mark
# &#35; — 数字标志 Number sign
$ &#36; — 美元标志 Dollar sign
% &#37; — 百分号 Percent sign
& &#38; &amp; Ampersand
‘ &#39; — 单引号 Apostrophe
( &#40; — 小括号左边部分 Left parenthesis
) &#41; — 小括号右边部分 Right parenthesis
* &#42; — 星号 Asterisk
+ &#43; — 加号 Plus sign
< &#60; &lt; 小于号 Less than
= &#61; — 等于符号 Equals sign
> &#62; &gt; 大于号 Greater than
? &#63; — 问号 Question mark
@ &#64; — Commercial at
[ &#91; --- 中括号左边部分 Left square bracket
\ &#92; --- 反斜杠 Reverse solidus (backslash)
] &#93; — 中括号右边部分 Right square bracket
{ &#123; — 大括号左边部分 Left curly brace
| &#124; — 竖线Vertical bar
} &#125; — 大括号右边部分 Right curly brace
```



