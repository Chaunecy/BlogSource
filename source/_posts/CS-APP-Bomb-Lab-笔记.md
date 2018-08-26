---
title: 'CS:APP - Bomb Lab 笔记'
date: 2018-08-22 17:41:33
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
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

```powershell
break function_name # 在函数入口处设置断点
break *0x4010b0     # 在此地址处设置断点
delete index        # 删除第 index 个断点，不加 index 参数即删除全部断点

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
```

> 打印寄存器中的内容时，可以先在该寄存器被赋值的指令处设置断点，否则得到的寄存器的值很可能是错的

## phase_1：字符串

```assembly
(gdb) disas phase_1
Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:		sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:		mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:		callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:	test   %eax,%eax
   0x0000000000400ef0 <+16>:	je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:	callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:	add    $0x8,%rsp
   0x0000000000400efb <+27>:	retq   
End of assembler dump.
```

