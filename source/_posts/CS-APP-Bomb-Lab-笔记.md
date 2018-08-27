---
title: 'CS:APP - Bomb Lab 笔记'
date: 2018-08-22 17:41:33
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
	- 学习笔记
---

## 调试工具介绍：GDB

使用以下命令进入 `gdb` 调试

```shell
gdb bomb        # bomb 为文件名
```

使用以下命令查看源代码

```shell
objdump -t bomb | less      # 使用 vim
objdump -d bomb > bomb.txt  # 导出源代码到 bomb.txt 文件
```

用到的 gdb 命令及介绍

<!-- more -->

```powershell
break function_name # 在函数入口处设置断点
break *0x4010b0     # 在此地址处设置断点
delete index        # 删除第 index 个断点，不加 index 参数即删除全部断点

run                 # 运行程序，记得先设断点

disas function_name # 反汇编函数，如果不加 function_name 就反汇编当前函数
disas *0xabcdffff   # 反汇编此地址附近的函数

stepi n             # 跳转到 n 条指令之后，如果不加 n 就跳转到下一条指令
next				# 运行到当前函数返回后的下一条指令，
                    # 如果是递归函数，则是进入递归树中下一个函数入口

# 以下命令中的寄存器都可替换成地址，如
# print 0x4010b0
print $esp          # 打印寄存器中的内容
print /x $esp       # 以 16 进制打印寄存器中的内容
x/s $esp            # 打印地址中的内容
print /x *(int *) $ebp # 以 16 进制打印地址中的内容，int 类型

set args ./sol.txt  # 以文件为输入，每次读取一行，避免重复输入
```

> 打印寄存器中的内容时，可以先在该寄存器被赋值的指令处设置断点，否则得到的寄存器的值很可能是错的

## phase_1：字符串

`run` 指令运行程序前，在 `explode_bomb` 及 `phase_1` 处设置断点。

执行 `run` 指令，程序进入 `phase_1` 函数入口处，使用 `disas` 进行反汇编。以下是第一关反汇编后的代码：

```assembly
(gdb) disas phase_1
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:	    sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:	    mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:	    callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq   
End of assembler dump.
```

第一行是在栈上为函数留出足够的空间

```assembly
   0x0000000000400ee0 <+0>:	    sub    $0x8,%rsp # stack pointer
```

第二行中，我们发现一个奇怪的东西被放进了 `%esi` 寄存器。来看看它是什么：

```shell
(gdb) stepi 2         # 运行两行机器代码，到 callq
0x0000000000400ee9 in phase_1 ()
# 输出内容
(gdb) print $esi
$1 = 4203520
(gdb) print *(int *) $esi
$2 = 1685221186
# 输出 %esi 寄存器中所在地址处的内容
(gdb) x/s $esi        
0x402400:	"Border relations with Canada have never been better."
# 输出一个字符
(gdb) print *(char *) $esi
$3 = 66 'B'
# 按 char * 类型输出内容
(gdb) print (char *) $esi 
$4 = 0x402400 "Border relations with Canada have never been better."
```

然后我们把 `Border relations with Canada have never been better.` 作为输入，即可通过第一关。

## phase_2：6 个数

先看代码：

```assembly
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
   0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     callq  0x40145c <read_six_numbers>
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    callq  0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:    add    $0x28,%rsp
---Type <return> to continue, or q <return> to quit---
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp
   0x0000000000400f42 <+70>:    retq   
End of assembler dump.
```

这次的代码长了不少，不过我们慢慢来看。

前 3 行保存调用者的信息，并为当前函数创建足够的栈帧：

```assembly
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
```

接下来有一个函数调用：

```assembly
   0x0000000000400f05 <+9>:     callq  0x40145c <read_six_numbers>
```

通过函数名我们知道是要读入 6 个数字。

> 说句题外话，这说明了函数命名的重要性，在为函数命名时一定要简洁而贴切！

然后就到了这里：

```assembly
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
```

它为什么要和 1 进行比较呢？`%rsp` 寄存器是指向栈顶的，加括号后即是取栈顶的值。在 32 位机器中，栈顶是保存函数参数的，在 64 位机器可能也是这样。所以我们大胆猜测，`(%rsp)` 中保存的，是我们输入的第一个值。

假设我们输入的是 `1 3 5 7 9 11`，那么应该有 `(%rsp)` 的值为 1。

验证一下，

```shell
Breakpoint 3, 0x0000000000400efc in phase_2 ()
# 运行到 read_six_numbers 入口处
(gdb) stepi 4
0x0000000000400f05 in phase_2 ()
(gdb) print *(int *) $rsp
$5 = -8632
(gdb) stepi
0x000000000040145c in read_six_numbers ()
# 跳出 read_six_numbers 函数
(gdb) next
Single stepping until exit from function read_six_numbers,
which has no line number information.
0x0000000000400f0a in phase_2 ()
# 就是这里，查看输入
(gdb) print *(int *) $rsp
$6 = 1
```

再下一行是根据比较的结果进行跳转，如果我们输入的第一个数字不是 1 的话，炸弹就会 Boom!!

接下来就比较简单了，取出第 2 个数字，然后把它的前一个数字乘以 2：

```assembly
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
```

比较它们是否相等。后面的第 3，4，5，6 个数字也是这样。

看到这里，我们知道，这是一个以 2 公比的等比数列，所以答案就是

```assembly
1 2 4 8 16 32
```

## phase_3：switch

```assembly
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:     sub    $0x18,%rsp
   0x0000000000400f47 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000400f4c <+9>:     lea    0x8(%rsp),%rdx
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmpq   *0x402470(,%rax,8)
   0x0000000000400f7c <+57>:    mov    $0xcf,%eax
   0x0000000000400f81 <+62>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:    mov    $0x2c3,%eax
   0x0000000000400f88 <+69>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:    mov    $0x100,%eax
   0x0000000000400f8f <+76>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:    mov    $0x185,%eax
   0x0000000000400f96 <+83>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:    mov    $0xce,%eax
---Type <return> to continue, or q <return> to quit---
   0x0000000000400f9d <+90>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:    mov    $0x2aa,%eax
   0x0000000000400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:    mov    $0x147,%eax
   0x0000000000400fab <+104>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:   callq  0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:   mov    $0x0,%eax
   0x0000000000400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:   mov    $0x137,%eax
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   retq   
End of assembler dump.
```

看到后面那么多的无条件跳转 `jmp` 了吗？与 switch 语句的代码非常相似。

这一关的答案似乎并不唯一，我是通过测试得出的结果。

首先还是这个可疑的操作：

```assembly
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
```

查看 `$esi` 处存储的内容。

```shell
Breakpoint 4, 0x0000000000400f43 in phase_3 ()
(gdb) print 0x4025cf
$7 = 4203983
# 要的就是它
(gdb) x/s 0x4025cf
0x4025cf:	"%d %d"
```

我们发现了类似于输入格式的东西，结合下面的代码，可以知道它应该就是输入格式。

```assembly
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   # %eax 存储输入的参数个数，说明参数要大于 1
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    callq  0x40143a <explode_bomb>
```

然后就是 switch 语句相关了。

```assembly
   # default 语句，boom
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmpq   *0x402470(,%rax,8)
```

其中 `8(%rsp)` 处存储的是第一个数字，至于为什么不是 `(%rsp)`，暂时没有想明白。

根据第一个数字进行跳转，查看 `0x402470(, %rax, 8)` 中的地址，可以知道 `jmpq` 的目的地址，然后计算第二个数字的大小，就可通关。

我的输入是

```assembly
1 311
```

## phase_4：递归函数

第四关是关于递归函数的。

```assembly
Dump of assembler code for function phase_4:
   0x000000000040100c <+0>:     sub    $0x18,%rsp
   0x0000000000401010 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000401015 <+9>:     lea    0x8(%rsp),%rdx
   0x000000000040101a <+14>:    mov    $0x4025cf,%esi
   0x000000000040101f <+19>:    mov    $0x0,%eax
   0x0000000000401024 <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000401029 <+29>:    cmp    $0x2,%eax
   0x000000000040102c <+32>:    jne    0x401035 <phase_4+41>
   0x000000000040102e <+34>:    cmpl   $0xe,0x8(%rsp)
   0x0000000000401033 <+39>:    jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:    callq  0x40143a <explode_bomb>
   0x000000000040103a <+46>:    mov    $0xe,%edx
   0x000000000040103f <+51>:    mov    $0x0,%esi
   0x0000000000401044 <+56>:    mov    0x8(%rsp),%edi
   0x0000000000401048 <+60>:    callq  0x400fce <func4>
   0x000000000040104d <+65>:    test   %eax,%eax
   0x000000000040104f <+67>:    jne    0x401058 <phase_4+76>
   0x0000000000401051 <+69>:    cmpl   $0x0,0xc(%rsp)
   0x0000000000401056 <+74>:    je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:    callq  0x40143a <explode_bomb>
   0x000000000040105d <+81>:    add    $0x18,%rsp
   0x0000000000401061 <+85>:    retq   
End of assembler dump.
```

首先还是熟悉的操作：

```assembly
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
```

这里我们要输入两个数字。其中第一个不能大于 0xe。第二个等于 0。

```assembly
   # 第一个数 <= 0xe
   0x000000000040102e <+34>:    cmpl   $0xe,0x8(%rsp)
   0x0000000000401033 <+39>:    jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:    callq  0x40143a <explode_bomb> 
   # 第二个数 == 0
   0x0000000000401051 <+69>:    cmpl   $0x0,0xc(%rsp)
```

然后是万恶的递归函数：

```assembly
   0x000000000040103a <+46>:    mov    $0xe,%edx
   0x000000000040103f <+51>:    mov    $0x0,%esi
   0x0000000000401044 <+56>:    mov    0x8(%rsp),%edi
   0x0000000000401048 <+60>:    callq  0x400fce <func4>
```

func4 的函数代码如下：

```assembly
Dump of assembler code for function func4:
   0x0000000000400fce <+0>:     sub    $0x8,%rsp
   0x0000000000400fd2 <+4>:     mov    %edx,%eax
   0x0000000000400fd4 <+6>:     sub    %esi,%eax
   0x0000000000400fd6 <+8>:     mov    %eax,%ecx
   0x0000000000400fd8 <+10>:    shr    $0x1f,%ecx
   0x0000000000400fdb <+13>:    add    %ecx,%eax
   0x0000000000400fdd <+15>:    sar    %eax
   0x0000000000400fdf <+17>:    lea    (%rax,%rsi,1),%ecx
   0x0000000000400fe2 <+20>:    cmp    %edi,%ecx
   0x0000000000400fe4 <+22>:    jle    0x400ff2 <func4+36>
   0x0000000000400fe6 <+24>:    lea    -0x1(%rcx),%edx
   0x0000000000400fe9 <+27>:    callq  0x400fce <func4>
   0x0000000000400fee <+32>:    add    %eax,%eax
   0x0000000000400ff0 <+34>:    jmp    0x401007 <func4+57>
   0x0000000000400ff2 <+36>:    mov    $0x0,%eax
   0x0000000000400ff7 <+41>:    cmp    %edi,%ecx
   0x0000000000400ff9 <+43>:    jge    0x401007 <func4+57>
   0x0000000000400ffb <+45>:    lea    0x1(%rcx),%esi
   0x0000000000400ffe <+48>:    callq  0x400fce <func4>
   0x0000000000401003 <+53>:    lea    0x1(%rax,%rax,1),%eax
   0x0000000000401007 <+57>:    add    $0x8,%rsp
   0x000000000040100b <+61>:    retq   
End of assembler dump.
```

别看它操作这么复杂，其实还是挺简单的。

```assembly
   0x0000000000400fd2 <+4>:     mov    %edx,%eax  # $eax = 0xe
   0x0000000000400fd4 <+6>:     sub    %esi,%eax  # $eax -= 0
   0x0000000000400fd6 <+8>:     mov    %eax,%ecx  # $ecx = 0xe
   0x0000000000400fd8 <+10>:    shr    $0x1f,%ecx # $ecx = 0
   0x0000000000400fdb <+13>:    add    %ecx,%eax  # $eax += 0
   0x0000000000400fdd <+15>:    sar    %eax       # $eax /= 2, $eax = 7
   # $ecx = $rax + $rsi = 7 + 0 = 7
   0x0000000000400fdf <+17>:    lea    (%rax,%rsi,1),%ecx
```

只要第一个值等于这个 7 就可以令 `%eax` 值设为 0，然后成功过关。

## phase_5：加密的字符串

这一关是第一关的加强版，仍然是输入一串字符串，但是有长度限制，而且是加密的！

可能因为有字符串读取，所以设置了金丝雀值。

```assembly
Dump of assembler code for function phase_5:
   0x0000000000401062 <+0>:     push   %rbx
   0x0000000000401063 <+1>:     sub    $0x20,%rsp
   0x0000000000401067 <+5>:     mov    %rdi,%rbx
   0x000000000040106a <+8>:     mov    %fs:0x28,%rax
   0x0000000000401073 <+17>:    mov    %rax,0x18(%rsp)
   0x0000000000401078 <+22>:    xor    %eax,%eax
   0x000000000040107a <+24>:    callq  0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000401089 <+39>:    jmp    0x4010d2 <phase_5+112>
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx
   0x000000000040108f <+45>:    mov    %cl,(%rsp)
   0x0000000000401092 <+48>:    mov    (%rsp),%rdx
   0x0000000000401096 <+52>:    and    $0xf,%edx
   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
   0x00000000004010ae <+76>:    movb   $0x0,0x16(%rsp)
   0x00000000004010b3 <+81>:    mov    $0x40245e,%esi
   0x00000000004010b8 <+86>:    lea    0x10(%rsp),%rdi
   0x00000000004010bd <+91>:    callq  0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:    test   %eax,%eax
   0x00000000004010c4 <+98>:    je     0x4010d9 <phase_5+119>
   0x00000000004010c6 <+100>:   callq  0x40143a <explode_bomb>
   0x00000000004010cb <+105>:   nopl   0x0(%rax,%rax,1)
   0x00000000004010d0 <+110>:   jmp    0x4010d9 <phase_5+119>
   0x00000000004010d2 <+112>:   mov    $0x0,%eax
   0x00000000004010d7 <+117>:   jmp    0x40108b <phase_5+41>
   0x00000000004010d9 <+119>:   mov    0x18(%rsp),%rax
   0x00000000004010de <+124>:   xor    %fs:0x28,%rax
   0x00000000004010e7 <+133>:   je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:   callq  0x400b30 <__stack_chk_fail@plt>
---Type <return> to continue, or q <return> to quit---
   0x00000000004010ee <+140>:   add    $0x20,%rsp
   0x00000000004010f2 <+144>:   pop    %rbx
   0x00000000004010f3 <+145>:   retq   
End of assembler dump.
```

输入的字符串长度必须是 6，否则爆炸：

```assembly
   0x0000000000401078 <+22>:    xor    %eax,%eax
   0x000000000040107a <+24>:    callq  0x40131b <string_length>
   0x000000000040107f <+29>:    cmp    $0x6,%eax
   0x0000000000401082 <+32>:    je     0x4010d2 <phase_5+112>
   0x0000000000401084 <+34>:    callq  0x40143a <explode_bomb>
```

浏览一下函数，是熟悉的操作：

```assembly
   0x00000000004010b3 <+81>:    mov    $0x40245e,%esi
```

寄存器的使用有关约定俗成的操作，所以这里放置的应该是与答案有关的东西。

```shell
(gdb) print (char *) 0x40245e
$15 = 0x40245e "flyers"
```

但是输入 `flyers` 并不能过关，因为输入的字符串被“加密了”。

加密相关代码如下：

```assembly
   0x000000000040108b <+41>:    movzbl (%rbx,%rax,1),%ecx
   0x000000000040108f <+45>:    mov    %cl,(%rsp)
   0x0000000000401092 <+48>:    mov    (%rsp),%rdx
   0x0000000000401096 <+52>:    and    $0xf,%edx
   0x0000000000401099 <+55>:    movzbl 0x4024b0(%rdx),%edx
   0x00000000004010a0 <+62>:    mov    %dl,0x10(%rsp,%rax,1)
   0x00000000004010a4 <+66>:    add    $0x1,%rax
   0x00000000004010a8 <+70>:    cmp    $0x6,%rax
   0x00000000004010ac <+74>:    jne    0x40108b <phase_5+41>
```

我们知道，`%rsp` 中会存储函数的参数，而 `<+62>` 那一行对 `%rsp` 中某些地址的值进行了修改！这里出现了又一个地址：`0x4024b0`。查看它的信息：

```shell
(gdb) print (char *) 0x4024b0
$16 = 0x4024b0 <array> "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```

看到了吗？它是一个字条数组，前面是 16 个字符，后面是挑衅。而为什么是 16 个字符？因为 `<+52>` 那里使用了 `and    $0xf,%edx` 操作呀！

然后我们可以依次输入 `abcdef`，`ghijkl`，`mnopqr` 等，来查看 26 个字母被映射后的值。

这里的答案并不唯一，我的输入是：

```shell
ionefg
```

## phase_6：排序

这一关很麻烦，但是并不难，但也用去了 2 个小时的时间。用笔纸记录下函数运行过程的数据会有很大帮助。

```assembly
Dump of assembler code for function phase_6:
   0x00000000004010f4 <+0>:     push   %r14
   0x00000000004010f6 <+2>:     push   %r13
   0x00000000004010f8 <+4>:     push   %r12
   0x00000000004010fa <+6>:     push   %rbp
   0x00000000004010fb <+7>:     push   %rbx
   0x00000000004010fc <+8>:     sub    $0x50,%rsp
   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    callq  0x40145c <read_six_numbers>
   0x000000000040110b <+23>:    mov    %rsp,%r14
   0x000000000040110e <+26>:    mov    $0x0,%r12d
   0x0000000000401114 <+32>:    mov    %r13,%rbp
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax
   0x000000000040111b <+39>:    sub    $0x1,%eax
   0x000000000040111e <+42>:    cmp    $0x5,%eax
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    callq  0x40143a <explode_bomb>
   0x0000000000401128 <+52>:    add    $0x1,%r12d
   0x000000000040112c <+56>:    cmp    $0x6,%r12d
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:    mov    %r12d,%ebx
   0x0000000000401135 <+65>:    movslq %ebx,%rax
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    callq  0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:    add    $0x4,%r13
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
---Type <return> to continue, or q <return> to quit---
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>
   0x000000000040116f <+123>:   mov    $0x0,%esi
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx
   0x000000000040117a <+134>:   add    $0x1,%eax
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   0x000000000040118d <+153>:   add    $0x4,%rsi
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx
   0x000000000040119a <+166>:   cmp    $0x1,%ecx
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>
   0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:   mov    %rbx,%rcx
   0x00000000004011bd <+201>:   mov    (%rax),%rdx
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)
   0x00000000004011c4 <+208>:   add    $0x8,%rax
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
   0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
   0x00000000004011da <+230>:   mov    $0x5,%ebp
---Type <return> to continue, or q <return> to quit---
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:   mov    (%rax),%eax
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   callq  0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
   0x00000000004011f7 <+259>:   add    $0x50,%rsp
   0x00000000004011fb <+263>:   pop    %rbx
   0x00000000004011fc <+264>:   pop    %rbp
   0x00000000004011fd <+265>:   pop    %r12
   0x00000000004011ff <+267>:   pop    %r13
   0x0000000000401201 <+269>:   pop    %r14
   0x0000000000401203 <+271>:   retq   
End of assembler dump.
```

函数太长了，我们分部分看。

第一部分：读取 6 个数字，每个数字不能大于 6，不能有重复的数

```assembly
   # 备份栈指针
   0x0000000000401100 <+12>:    mov    %rsp,%r13
   0x0000000000401103 <+15>:    mov    %rsp,%rsi
   0x0000000000401106 <+18>:    callq  0x40145c <read_six_numbers>
   0x000000000040110b <+23>:    mov    %rsp,%r14
   0x000000000040110e <+26>:    mov    $0x0,%r12d
   0x0000000000401114 <+32>:    mov    %r13,%rbp
   # 读取数字
   0x0000000000401117 <+35>:    mov    0x0(%r13),%eax
   0x000000000040111b <+39>:    sub    $0x1,%eax
   # 不能大于 5，但因为减去了 1，所以是不能大于 6
   0x000000000040111e <+42>:    cmp    $0x5,%eax
   0x0000000000401121 <+45>:    jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:    callq  0x40143a <explode_bomb>
   # %r12d 记录当前进度，因为是对 6 个数依次遍历，所以是记录当前
   # 运行到第几个数了
   0x0000000000401128 <+52>:    add    $0x1,%r12d
   0x000000000040112c <+56>:    cmp    $0x6,%r12d
   0x0000000000401130 <+60>:    je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:    mov    %r12d,%ebx
   # 当前运行到的数，与后面的数不能有重复的
   0x0000000000401135 <+65>:    movslq %ebx,%rax
   0x0000000000401138 <+68>:    mov    (%rsp,%rax,4),%eax
   0x000000000040113b <+71>:    cmp    %eax,0x0(%rbp)
   0x000000000040113e <+74>:    jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:    callq  0x40143a <explode_bomb>
   0x0000000000401145 <+81>:    add    $0x1,%ebx
   0x0000000000401148 <+84>:    cmp    $0x5,%ebx
   0x000000000040114b <+87>:    jle    0x401135 <phase_6+65>
   # 依次读取下一个数
   0x000000000040114d <+89>:    add    $0x4,%r13
   0x0000000000401151 <+93>:    jmp    0x401114 <phase_6+32>
```

总结来看，就是要读取 6 个数字，这 6 个数字各不相同，且小于等于 6

第二部分：`num = 7 - num`

```assembly
   0x0000000000401153 <+95>:    lea    0x18(%rsp),%rsi
   0x0000000000401158 <+100>:   mov    %r14,%rax
   # 取 7
   0x000000000040115b <+103>:   mov    $0x7,%ecx
   0x0000000000401160 <+108>:   mov    %ecx,%edx
---Type <return> to continue, or q <return> to quit---
   # 用 7 减去输入的值
   0x0000000000401162 <+110>:   sub    (%rax),%edx
   0x0000000000401164 <+112>:   mov    %edx,(%rax)
   0x0000000000401166 <+114>:   add    $0x4,%rax
   0x000000000040116a <+118>:   cmp    %rsi,%rax
   0x000000000040116d <+121>:   jne    0x401160 <phase_6+108>
```

第三部分：分配地址

```assembly
   0x000000000040116f <+123>:   mov    $0x0,%esi
   0x0000000000401174 <+128>:   jmp    0x401197 <phase_6+163>
   # 这里根据当前值的大小，决定更新几次。因为每个值都不一样
   # 所以最终每个值对应的地址也不一样
   0x0000000000401176 <+130>:   mov    0x8(%rdx),%rdx
   0x000000000040117a <+134>:   add    $0x1,%eax
   0x000000000040117d <+137>:   cmp    %ecx,%eax
   0x000000000040117f <+139>:   jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:   jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:   mov    $0x6032d0,%edx
   # 把地址存到栈中，第一个地址存在 $rsp + 0x20 处
   0x0000000000401188 <+148>:   mov    %rdx,0x20(%rsp,%rsi,2)
   # 开始找下一个值对应的地址，超过 6 就进入下一部分
   0x000000000040118d <+153>:   add    $0x4,%rsi
   0x0000000000401191 <+157>:   cmp    $0x18,%rsi
   0x0000000000401195 <+161>:   je     0x4011ab <phase_6+183>
   # 获取当前值的大小，如果当前值是 1，就直接赋值为 0x6032d0
   0x0000000000401197 <+163>:   mov    (%rsp,%rsi,1),%ecx
   0x000000000040119a <+166>:   cmp    $0x1,%ecx
   0x000000000040119d <+169>:   jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:   mov    $0x1,%eax
   # 关键就在这里，分配一个初始地址，进行更新
   0x00000000004011a4 <+176>:   mov    $0x6032d0,%edx
   0x00000000004011a9 <+181>:   jmp    0x401176 <phase_6+130>
```

这里的操作是：按照输入值的大小，给每个输入的值分配一个地址。

如果把地址按无符号整数进行排序，那么可以形成一个以 0x10 为公差的等差数列。

第四部分：生成指针

```assembly
   # 第一个地址
   0x00000000004011ab <+183>:   mov    0x20(%rsp),%rbx
   # 第二个地址
   0x00000000004011b0 <+188>:   lea    0x28(%rsp),%rax
   # 循环结束标志
   0x00000000004011b5 <+193>:   lea    0x50(%rsp),%rsi
   0x00000000004011ba <+198>:   mov    %rbx,%rcx
   0x00000000004011bd <+201>:   mov    (%rax),%rdx
   # 因为地址之间最小相差 0x10，所以加上 0x8 后是不会覆盖
   # 原来的值的
   0x00000000004011c0 <+204>:   mov    %rdx,0x8(%rcx)
   0x00000000004011c4 <+208>:   add    $0x8,%rax
   0x00000000004011c8 <+212>:   cmp    %rsi,%rax
   0x00000000004011cb <+215>:   je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:   mov    %rdx,%rcx
   0x00000000004011d0 <+220>:   jmp    0x4011bd <phase_6+201>
   # 最后一个指向 0
   0x00000000004011d2 <+222>:   movq   $0x0,0x8(%rdx)
```

这一部分运行完成后，会形成以下效果：

```markdown
addr1        addr2        addr3        addr4        addr5        addr6        0
    addr1+8 /      addr2+8/     addr3+8/     addr4+8/     addr5+8/    addr6+8/
```

这应该是为了方便后面进行比较（？）吧。

第五部分：比较大小

```assembly
   # 一共 6 个数，比较 5 次
   0x00000000004011da <+230>:   mov    $0x5,%ebp
---Type <return> to continue, or q <return> to quit---
   # 获取它的后一个地址，根据第四部分
   0x00000000004011df <+235>:   mov    0x8(%rbx),%rax
   0x00000000004011e3 <+239>:   mov    (%rax),%eax
   # 比较当前地址存储的值 ($rbx) 与 下一个地址存储的值
   0x00000000004011e5 <+241>:   cmp    %eax,(%rbx)
   # 当前地址存储的值更大
   0x00000000004011e7 <+243>:   jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:   callq  0x40143a <explode_bomb>
   0x00000000004011ee <+250>:   mov    0x8(%rbx),%rbx
   0x00000000004011f2 <+254>:   sub    $0x1,%ebp
   0x00000000004011f5 <+257>:   jne    0x4011df <phase_6+235>
```

根据每个地址存储的值的大小，从大到小进行排序，就能保证成功通过这一部分。

但是，在第二部分中我们使用了 `num = 7 - num`，所以还要多一步操作。

最终答案是：

```
4 3 2 1 6 5
```

## secret_phase：递归与比较

> secret_phase 的进入参考了 [小土刀](https://wdxtub.com/2016/04/16/thick-csapp-lab-2/) 的博客。

完成 phase_6 后，程序就结束了，但是，看一下程序“源代码”，发现了这么一行注释：

```  
    /* Wow, they got it!  But isn't something... missing?  Perhaps
     * something they overlooked?  Mua ha ha ha ha! */
```

然后查找汇编源代码，发现有 `secret_phase` 这么一个函数，在 `phase_defused` 函数中被调用。

先看一下 `phase_defused` 的代码：

```assembly
Dump of assembler code for function phase_defused:
   0x00000000004015c4 <+0>:     sub    $0x78,%rsp
   0x00000000004015c8 <+4>:     mov    %fs:0x28,%rax
   0x00000000004015d1 <+13>:    mov    %rax,0x68(%rsp)
   0x00000000004015d6 <+18>:    xor    %eax,%eax
   # 过了 6 关了没，没有就直接结束了
   0x00000000004015d8 <+20>:    cmpl   $0x6,0x202181(%rip)        # 0x603760 <num_input_strings>
   0x00000000004015df <+27>:    jne    0x40163f <phase_defused+123>
   0x00000000004015e1 <+29>:    lea    0x10(%rsp),%r8
   0x00000000004015e6 <+34>:    lea    0xc(%rsp),%rcx
   0x00000000004015eb <+39>:    lea    0x8(%rsp),%rdx
   # 多么熟悉的操作！！
   0x00000000004015f0 <+44>:    mov    $0x402619,%esi
   0x00000000004015f5 <+49>:    mov    $0x603870,%edi
   0x00000000004015fa <+54>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x00000000004015ff <+59>:    cmp    $0x3,%eax
   0x0000000000401602 <+62>:    jne    0x401635 <phase_defused+113>
   0x0000000000401604 <+64>:    mov    $0x402622,%esi
   0x0000000000401609 <+69>:    lea    0x10(%rsp),%rdi
   0x000000000040160e <+74>:    callq  0x401338 <strings_not_equal>
   0x0000000000401613 <+79>:    test   %eax,%eax
   # 如果字符串不一样，就结束了
   0x0000000000401615 <+81>:    jne    0x401635 <phase_defused+113>
   # 进入正戏！
   0x0000000000401617 <+83>:    mov    $0x4024f8,%edi
   0x000000000040161c <+88>:    callq  0x400b10 <puts@plt>
   0x0000000000401621 <+93>:    mov    $0x402520,%edi
   0x0000000000401626 <+98>:    callq  0x400b10 <puts@plt>
   0x000000000040162b <+103>:   mov    $0x0,%eax
   0x0000000000401630 <+108>:   callq  0x401242 <secret_phase>
   0x0000000000401635 <+113>:   mov    $0x402558,%edi
   0x000000000040163a <+118>:   callq  0x400b10 <puts@plt>
   0x000000000040163f <+123>:   mov    0x68(%rsp),%rax
   0x0000000000401644 <+128>:   xor    %fs:0x28,%rax
   0x000000000040164d <+137>:   je     0x401654 <phase_defused+144>
   0x000000000040164f <+139>:   callq  0x400b30 <__stack_chk_fail@plt>
   0x0000000000401654 <+144>:   add    $0x78,%rsp
   0x0000000000401658 <+148>:   retq   
End of assembler dump.
```

看到那熟悉的操作了吗？下面来看看它们存储的内容是什么：

```shell
# 这里是输入格式
(gdb) x/s $esi
0x402619:	"%d %d %s"

# 这里是读入输入内容，这里我已经完成了，所以是以下结果
# 根据 "7 0" 可以看出是第 4 关
(gdb) x/s $edi
0x603870 <input_strings+240>:	"7 0 DrEvil"

# 以下可以跳过
(gdb) stepi
0x0000000000400bf0 in __isoc99_sscanf@plt ()
(gdb) next
Single stepping until exit from function __isoc99_sscanf@plt,
which has no line number information.
__isoc99_sscanf (s=0x603870 <input_strings+240> "7 0 DrEvil", format=0x402619 "%d %d %s") at isoc99_sscanf.c:26
26	isoc99_sscanf.c: 没有那个文件或目录.
(gdb) next
30	in isoc99_sscanf.c
(gdb) next
31	in isoc99_sscanf.c
(gdb) next
30	in isoc99_sscanf.c
(gdb) next
31	in isoc99_sscanf.c
(gdb) next
35	in isoc99_sscanf.c
(gdb) next
0x00000000004015ff in phase_defused ()
(gdb) stepi 3
0x0000000000401609 in phase_defused ()

# 下面要进行字符串比较了
# $esi 是被用于比较的，$rsp + 0x10 存储的是你的输入。
(gdb) x/s $esi
0x402622:	"DrEvil"
(gdb) print (char *) ($rsp + 0x10)
$29 = 0x7fffffffdcf0 "DrEvil"
```

> 至于为什么在第四关输入了 3 个参数而没有引发 boom，可能是因为 __isoc99_sscanf 函数有关吧。
>
> phase_4 中 %edi 用于存储输入参数。

接下来就是进入 secret_phase 的代码了，先看看其中的地址存储了什么：

```shell
(gdb) x/s 0x4024f8
0x4024f8:	"Curses, you've found the secret phase!"
(gdb) x/s 0x402520
0x402520:	"But finding it and solving it are quite different..."
(gdb) x/s 0x402558
0x402558:	"Congratulations! You've defused the bomb!"
```

嗯，没什么有价值的东西。直接看 `secret_phase` 吧：

```assembly
# 就是上面的挑衅
Curses, you've found the secret phase!
But finding it and solving it are quite different...

Breakpoint 7, 0x0000000000401242 in secret_phase ()
(gdb) disas
Dump of assembler code for function secret_phase:
=> 0x0000000000401242 <+0>:     push   %rbx
   0x0000000000401243 <+1>:     callq  0x40149e <read_line>
   0x0000000000401248 <+6>:     mov    $0xa,%edx
   0x000000000040124d <+11>:    mov    $0x0,%esi
   0x0000000000401252 <+16>:    mov    %rax,%rdi
   0x0000000000401255 <+19>:    callq  0x400bd0 <strtol@plt>
   0x000000000040125a <+24>:    mov    %rax,%rbx
   0x000000000040125d <+27>:    lea    -0x1(%rax),%eax
   0x0000000000401260 <+30>:    cmp    $0x3e8,%eax
   0x0000000000401265 <+35>:    jbe    0x40126c <secret_phase+42>
   0x0000000000401267 <+37>:    callq  0x40143a <explode_bomb>
   0x000000000040126c <+42>:    mov    %ebx,%esi
   0x000000000040126e <+44>:    mov    $0x6030f0,%edi
   0x0000000000401273 <+49>:    callq  0x401204 <fun7>
   0x0000000000401278 <+54>:    cmp    $0x2,%eax
   0x000000000040127b <+57>:    je     0x401282 <secret_phase+64>
   0x000000000040127d <+59>:    callq  0x40143a <explode_bomb>
   0x0000000000401282 <+64>:    mov    $0x402438,%edi
   0x0000000000401287 <+69>:    callq  0x400b10 <puts@plt>
   0x000000000040128c <+74>:    callq  0x4015c4 <phase_defused>
   0x0000000000401291 <+79>:    pop    %rbx
   0x0000000000401292 <+80>:    retq   
End of assembler dump.
```

看到函数这么短，是不是松了一口气？先别急，看到 `callq <fun7>` 这一行了没？大头在这里呢！

先看秘密关卡的代码逻辑，它会先读取一行输入，把输入的字符串转换成 10 进制的长整型，然后这个数字还要减 1 后与 0x3e8 进行比较。

然后是关键部分：

```assembly
   0x000000000040126c <+42>:    mov    %ebx,%esi
   0x000000000040126e <+44>:    mov    $0x6030f0,%edi
   0x0000000000401273 <+49>:    callq  0x401204 <fun7>
   0x0000000000401278 <+54>:    cmp    $0x2,%eax
   0x000000000040127b <+57>:    je     0x401282 <secret_phase+64>
   0x000000000040127d <+59>:    callq  0x40143a <explode_bomb>
```

在 `fun7` 入口处设置断点，查看寄存器内的值：

```assembly
(gdb) print *(int *) $edi
$36 = 36

# 这是是我的输入，转化成长整型后
(gdb) print $esi
$37 = 22
```

这样，我们就知道，在输入值为 `$esi` 的情况下 `fun7` 的返回值 `$eax` 要为 2，才能通过秘密关卡。

再看 `fun7` 中进行了什么操作，才能使得 `%eax` 的值被设置为 2。

```assembly
Dump of assembler code for function fun7:
=> 0x0000000000401204 <+0>:     sub    $0x8,%rsp
   0x0000000000401208 <+4>:     test   %rdi,%rdi
   0x000000000040120b <+7>:     je     0x401238 <fun7+52>
   0x000000000040120d <+9>:     mov    (%rdi),%edx
   0x000000000040120f <+11>:    cmp    %esi,%edx
   0x0000000000401211 <+13>:    jle    0x401220 <fun7+28>
   0x0000000000401213 <+15>:    mov    0x8(%rdi),%rdi
   0x0000000000401217 <+19>:    callq  0x401204 <fun7>
   0x000000000040121c <+24>:    add    %eax,%eax
   0x000000000040121e <+26>:    jmp    0x40123d <fun7+57>
   0x0000000000401220 <+28>:    mov    $0x0,%eax
   0x0000000000401225 <+33>:    cmp    %esi,%edx
   0x0000000000401227 <+35>:    je     0x40123d <fun7+57>
   0x0000000000401229 <+37>:    mov    0x10(%rdi),%rdi
   0x000000000040122d <+41>:    callq  0x401204 <fun7>
   0x0000000000401232 <+46>:    lea    0x1(%rax,%rax,1),%eax
   0x0000000000401236 <+50>:    jmp    0x40123d <fun7+57>
   0x0000000000401238 <+52>:    mov    $0xffffffff,%eax
   0x000000000040123d <+57>:    add    $0x8,%rsp
   0x0000000000401241 <+61>:    retq   
End of assembler dump.
```

又是一个递归函数，这里我把它转化成伪代码的样式，看起来轻松一些：

```pseudocode
fun7($edi, $esi):
	# $esi <- my input.
	
	$edx <- ($rdi)
	
	# my input is less than $edx
	if $edx > $esi
		# number 1
		$rdi <- M[$rdi + 0x8]
		call fun7($edi, $esi)
		$eax *= 2
	else
		$eax <- 0
		# number 2
		if $esi == $edx
			return
		else # number 3
			$rdi <- M[$rdi + 0x10]
			call fun7($edi, $esi)
			$eax = $rax * 2 + 1
```

> `M[rA]` 表示取地址 `rA` 中存放的值。

通过上述伪代码，我们知道，当输入值比较大时，会更新 `$rdi` 中存放的地址，并进入下一层递归，结束递归后，对 `$eax` 的值进行加倍操作；当输入值比较小时，也会更新 `$rdi` 中存放的地址，并进入下一层递归，结束递归后，对 `$eax` 的值进行加倍后再 +1 操作；只有当输入值 == `$rdi` 地址中存放的值时，才会结束递归，并为 `secret_phase` 返回 `$eax`。

那么，答案就很简单了，只要我们先进入分支 `number 1`，再进入分支 `number 3`，最后进入分支 `number 2`，就能依次得到：

```markdown
$eax <- 0                      # number 2
$eax <- $rax + $rax + 1 = 1    # number 3
$eax <= $eax + $eax = 2        # number 1
```

只要找到这样的值，它在进入 `number 2` 分支时与当时的 `M[$rdi]` 相同，即可通过此关！

最终答案即是：

```assembly
22
```

> 每次更新 `$rdi` 时，都重新计算了它的地址，所以没有计算的捷径，只能依次计算。使用纸笔记录下来中间结果会有很大帮助。

## 结语

Bomb Lab 到此就告一段落了，我们最终粉碎了邪恶博士的阴谋（捂脸）！

总得来说，十分贴合教材内容，关于跳转、switch、函数调用、递归调用都有所涉及，是一个不可多得的锻炼的机会。

```shell
(gdb) run
Starting program: /home/cwwang/文档/csapp/lab2/bomb/bomb ./sol.txt 
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
Wow! You've defused the secret stage!
Congratulations! You've defused the bomb!
[Inferior 1 (process 62328) exited normally]
```

以下是所有输入，放置在 `sol.txt` 文件中：

```markdown
Border relations with Canada have never been better.
1 2 4 8 16 32
1 311
7 0 DrEvil
ionefg
4 3 2 1 6 5
22
```

