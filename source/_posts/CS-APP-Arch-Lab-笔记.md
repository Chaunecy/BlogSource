---
title: 'CS: APP - Arch Lab 笔记'
date: 2018-09-02 20:08:19
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
	- 学习笔记
---

在这个 lab 中，我们会学习如何设计与实现流水线化的 Y86-64 处理器，优化并最大化示例程序性能。 

## 准备活动

在 `sim/` 目录下执行以下命令时：

``` bash
$ make clean
$ make
```

如果报错，可能是没有安装依赖： `flex` 和 `bison` 。

在任意目录下使用以下命令安装：

```bash
$ sudo apt-get install flex
> type in your password here...
$ sudo apt-get install bison
> type in your password here...
```

<!-- more -->

如果依然报错，则可能是没有 `tcl` 和 `tk`。这两个是用于 GUI 界面 debug 的。没有也无妨，博主就是人肉 debug 过来的。只要注释掉当前目录下的 `makefile` 文件中的内容：

```makefile
#GUIMODE=-DHAS_GUI
#TKLIBS=-L/usr/lib -ltk -ltcl
#TKINC=-isystem /usr/include
```

## Part A

第一部分是手写 Y86-64 代码。工作目录是 `sim/misc/`。要自己新建两个文件，分别是 `sum.ys`，`copy.ys`，实现要求的三个函数，分别是`sum_list`，`rsum_list`，`copy_block`。

如何测试代码：

```bash
# you are now at sim/misc
$ ./yas xxx.ys
$ ./yis xxx.yo
> %rax 0x0 -> 0xcba
> ... ...
```

只要执行 `./yas xxx.ys` 后没有报错，执行 `./yis xxx.yo` 后 `%rax = 0xcba` 就行了。 

Part A 只要如实翻译 c 代码即可，技术含量并不高，只是有点玄学。关于 `Y86-64` 的语法，除了后缀由 `irmovl` 变为 `irmovq` 外，其他都没有变。 

以下为 `sum_list` 与 `rsum_list` 的代码：

```assembly
# Execution begins at address 0 
	.pos 0
	irmovq stack, %rsp  	# Set up stack pointer
	call main		# Execute main program
	halt			# Terminate program 

# Sample linked list
	.align 8
ele1:
	.quad 0x00a
	.quad ele2
ele2:
	.quad 0x0b0
	.quad ele3
ele3:
	.quad 0xc00
	.quad 0	

main:
	irmovq ele1, %rdi
	call rsum_list
	ret


sum_list:
	xorq %rax, %rax
	jmp test
loop:
	mrmovq (%rdi), %rbx
	mrmovq 0x8(%rdi), %rdi
	addq %rbx, %rax
test:
	andq %rdi, %rdi
	jne loop
	ret

	
rsum_list:
	pushq %rcx              # 調用者保存
	xorq %rax, %rax         # result = 0
	andq %rdi, %rdi         # %rdi 是否爲 0，即是否完成遍历
	je end                  # 直接結束
	mrmovq (%rdi), %rcx     # 取得當前鏈表中節點的：值
	mrmovq 0x8(%rdi), %rdi  # 取得下一個節點的：地址
	call rsum_list          # 遞歸調用
	addq %rcx, %rax         # 相加
end:
	popq %rcx               # 調用者保存
	ret


# Stack starts here and grows to lower addresses
	.pos 0x200
stack:
```

这里多说一句，关于参数传递，是有着比较明确的标准的：

| 操作数大小 \ 参数数量 |   1    |   2    |   3    |   4    |   5   |   6   |
| :-------------------: | :----: | :----: | :----: | :----: | :---: | :---: |
|          64           | `%rdi` | `%rsi` | `%rdx` | `%rcx` | `%r8` | `%r9` |
|          32           | `%edi` | `%esi` | `%edx` | `%ecx` | `%r8d` | `r9d`      |
|          16           | `%di` | `%si` | `%dx` | `%cx` | `%r8w` | `r9w`      |
|           8           | `%dil` | `%sil` | `%dl` | `%cl` | `%r8b` | `r9b`      |

以下为 `copy_block` 的代码，注意参数传递的要求：

```assembly
	.pos 0
	irmovq stack, %rsp
	call main
	halt

	.align 8
# Source block
src:
	.quad 0x00a
	.quad 0x0b0
	.quad 0xc00
	
# Destination block
dest:
	.quad 0x111
	.quad 0x222
	.quad 0x333

	
main:
	irmovq src, %rdi
	irmovq dest, %rsi
	irmovq $0x3, %rdx
	call copy_block
	ret


copy_block:
	xorq %rax, %rax				# result = 0
	irmovq $0x8, %rcx           # src 與 dest 的自增
	irmovq $0x1, %r8            # len 的自增
	jmp test                    # 判斷是否結束循環
loop:
	mrmovq (%rdi), %rbx         # val = *src
	addq %rcx, %rdi             # src++
	rmmovq %rbx, (%rsi)         # *dest = val
	addq %rcx, %rsi             # dest++
	xorq %rbx, %rax             # result ^= val
	subq %r8, %rdx              # len--
test:
	andq %rdx, %rdx             # len > 0?
	jg loop
end:
	ret


	.pos 0x200
stack:
```

## Part B

第二部分要求实现 `iaddq`，修改 `seq-full.hcl` 文件，工作目录在 `sim/seq`。

如何测试你的代码：

```bash
#############################################
## you are now at archlab-handout/sim
#############################################
$ make VERSION=full
$ cd seq/

#############################################
## you are now at archlab-handout/sim/seq
#############################################
$ ./ssim -t ../y86-code/asumi.yo
> ISA Check Succeeds

$ cd ../y86-code
#############################################
## you are now at archlab-handout/sim/y86-code
#############################################
$ make testssim
> (Many) ISA Check Succeeds

$ cd ../ptest/
#############################################
## you are now at archlab-handout/sim/ptest 
#############################################
$ make SIM=../seq/ssim
> (Many) All xxx ISA Checks Succeed
$ make SIM=../seq/ssim TFLAGS=-i
```

上述过程全部 `Secceed` 就算完成了这个部分。

接下来是解题过程。

`iaddq` 其实相当于 `irmovq` 与 `addq` 的结合。先来看它的顺序实现：

|   stage    |                         iaddq V, rB                          |
| :--------: | :----------------------------------------------------------: |
|   fetch    | _icode: ifun <-- $M_1[PC]$<br />rA: rB <-- $M_1[PC + 1]$<br />valC <-- $M_4[PC + 2]$<br />valP <-- PC + 6 |
|   decode   |                        valB <-- R[rB]                        |
|  execute   |               valE <-- valB + valC<br />Set CC               |
|   memory   |                                                              |
| write back |                        R[rB] <-- valE                        |
| PC update  |                         PC <-- valP                          |

然后我们就可以修改 `seq-full.hcl` 文件了：

```hcl
bool instr_valid = icode in 
	{ INOP, IHALT, IRRMOVQ, IIRMOVQ, IRMMOVQ, IMRMOVQ,
	       IOPQ, IJXX, ICALL, IRET, IPUSHQ, IPOPQ, IIADDQ };

# Does fetched instruction require a regid byte?
bool need_regids =
	icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
		     IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ };

# Does fetched instruction require a constant word?
bool need_valC =
	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL, IIADDQ };

## What register should be used as the B source?
word srcB = [
	icode in { IOPQ, IRMMOVQ, IMRMOVQ, IIADDQ  } : rB;
	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't need register
];

## What register should be used as the E destination?
word dstE = [
	icode in { IRRMOVQ } && Cnd : rB;
	icode in { IIRMOVQ, IOPQ, IIADDQ} : rB;
	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't write any register
];

## Select input A to ALU
word aluA = [
	icode in { IRRMOVQ, IOPQ } : valA;
	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ } : valC;
	icode in { ICALL, IPUSHQ } : -8;
	icode in { IRET, IPOPQ } : 8;
	# Other instructions don't need ALU
];

## Select input B to ALU
word aluB = [
	icode in { IRMMOVQ, IMRMOVQ, IOPQ, ICALL, 
		      IPUSHQ, IRET, IPOPQ, IIADDQ } : valB;
	icode in { IRRMOVQ, IIRMOVQ } : 0;
	# Other instructions don't need ALU
];


## Should the condition codes be updated?
bool set_cc = icode in { IOPQ, IIADDQ };

```

## Part C

第三部分说难也难，说简单也简单。如果你想得高分，那它是真的难。

工作目录在 `sim/pipe`。只要求修改 `ncopy.ys` 和 `pipe.full.hcl`。

先看怎么检查答案的正确性：

```bash
## you are now at sim/pipe
$ make driver
$ ./correctness
```

查看得分：

```bash
$ ./benchmark.pl
```

修改 `pipe-full.hcl` 文件后：

```bash
## you are now at sim/pipe
$ make VERSION=full
$ cd ../y86-code/
$ make testpsim
$ cd ../ptest/
$ make SIM=../pipe/psim TFLAGS=-i
$ cd ../pipe/
$ ./correctness.pl -p
```

要全都 `Succeed` 成绩才算有效。

首先给出了示例程序 `ncopy`，复制数组并统计大于 0 的元素的个数。

```
/*
* ncopy - copy src to dst, returning number of positive ints
* contained in src array.
*/
word_t ncopy(word_t *src, word_t *dst, word_t len) {
	word_t count = 0;
	word_t val;
	while (len > 0) {
		val = *src++;
		*dst++ = val;
		if (val > 0)
			count++;
		len--;
	}
	return count;
}
```

其 `Y86-64` 代码如下：

```
# Do not modify this portion
# Function prologue.
# %rdi = src, %rsi = dst, %rdx = len
ncopy:

##################################################################
# You can modify this portion
	# Loop header
	xorq %rax,%rax		# count = 0;
	andq %rdx,%rdx		# len <= 0?
	jle Done		# if so, goto Done:

Loop:	mrmovq (%rdi), %r10	# read val from src...
	rmmovq %r10, (%rsi)	# ...and store it to dst
	andq %r10, %r10		# val <= 0?
	jle Npos		# if so, goto Npos:
	irmovq $1, %r10
	addq %r10, %rax		# count++
Npos:	irmovq $1, %r10
	subq %r10, %rdx		# len--
	irmovq $8, %r10
	addq %r10, %rdi		# src++
	addq %r10, %rsi		# dst++
	andq %rdx,%rdx		# len > 0?
	jg Loop			# if so, goto Loop:
```

总结我的优化历程，如下所示：

- 将条件控制转移改为条件数据传送，CPE 减 1
- 将 load/use 冒险去除，CPE 减 1
- 使用 loop unrolling 进行二路展开
- 实现 `iaddq`，因为其设置条件码，所以可以节省比较
- 配合 `iaddq`，去除不必要的指令 `irmovq`。有 0.7 分
- 改进跳转语句，对于常见的情况，优先跳转
- 使用 `mrmovq 0xXX(%rdi), %rbx` 这样的指令（捂脸）
- 先 8 路展开，剩下的尝试 4 路展开，达到 648 byte
- 对 `len = 1` 进行特别处理，从 37 降到 21，分数达到历史最高 38.9

现在的瓶颈出在小数据量部分，后续如果要继续优化，肯定要对它们进行处理。

下面是最终版本实现：

```assembly
# You can modify this portion
	# Loop header
	xorq %rax,%rax		# count = 0;
	iaddq $-1, %rdx     # 對 len = 1 的優化
	jne More
	mrmovq (%rdi), %r10
	irmovq $1, %r11
	rmmovq %r10, (%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	jmp Done
More:                   # 對不等於 1 的
	iaddq $-6, %rdx     # 一個副作用，小於 7 的直接進入 Last
	jg Loop
ForLessThan7:
	iaddq $0x3, %rdx
	jge Last
	iaddq $0x4, %rdx
	jne Fin
	jmp Done
Fin:                    # 剩下的不到 4 個
	mrmovq (%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, (%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	iaddq $0x8, %rdi
	iaddq $0x8, %rsi
	iaddq $-1, %rdx
	jg Fin
	jmp Done
Last:                    # 剩下的有 4 個及以上
	mrmovq (%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, (%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x8(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x8(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax

	mrmovq 0x10(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x10(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x18(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x18(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	iaddq $0x20, %rdi
	iaddq $0x20, %rsi
	andq %rdx, %rdx
	jg Fin
	jmp Done
Loop:                   # 大於 7 個
	mrmovq (%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, (%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x8(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x8(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax

	mrmovq 0x10(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x10(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x18(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x18(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x20(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x20(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x28(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x28(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax

	mrmovq 0x30(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x30(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	
	mrmovq 0x38(%rdi), %r10
	rrmovq %rax, %r11
	iaddq $0x1, %r11
	rmmovq %r10, 0x38(%rsi)
	andq %r10, %r10
	cmovg %r11, %rax
	iaddq $0x40, %rdi
	iaddq $0x40, %rsi
	
	iaddq $-8, %rdx
Test:
	jg Loop			# if so, goto Loop:
	jmp ForLessThan7

```

对 `piep-full.hcl` 的修改：

``` hcl

# Is instruction valid?
bool instr_valid = f_icode in 
	{ INOP, IHALT, IRRMOVQ, IIRMOVQ, IRMMOVQ, IMRMOVQ,
	  IOPQ, IJXX, ICALL, IRET, IPUSHQ, IPOPQ, IIADDQ };

# Does fetched instruction require a regid byte?
bool need_regids =
	f_icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
		     IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ };

# Does fetched instruction require a constant word?
bool need_valC =
	f_icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL, IIADDQ };

## What register should be used as the B source?
word d_srcB = [
	D_icode in { IOPQ, IRMMOVQ, IMRMOVQ, IIADDQ  } : D_rB;
	D_icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't need register
];

## What register should be used as the E destination?
word d_dstE = [
	D_icode in { IRRMOVQ, IIRMOVQ, IOPQ, IIADDQ} : D_rB;
	D_icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
	1 : RNONE;  # Don't write any register
];

## Select input A to ALU
word aluA = [
	E_icode in { IRRMOVQ, IOPQ } : E_valA;
	E_icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ } : E_valC;
	E_icode in { ICALL, IPUSHQ } : -8;
	E_icode in { IRET, IPOPQ } : 8;
	# Other instructions don't need ALU
];

## Select input B to ALU
word aluB = [
	E_icode in { IRMMOVQ, IMRMOVQ, IOPQ, ICALL, 
		     IPUSHQ, IRET, IPOPQ, IIADDQ } : E_valB;
	E_icode in { IRRMOVQ, IIRMOVQ } : 0;
	# Other instructions don't need ALU
];

## Should the condition codes be updated?
bool set_cc = E_icode in {IOPQ, IIADDQ} &&
	# State changes only during normal operation
	!m_stat in { SADR, SINS, SHLT } && !W_stat in { SADR, SINS, SHLT };

```

## 总结

想要完成这个 lab，至少要对第 4 章有一定的了解，通读一遍，并做完配有答案的习题。再学习一下第 5.8 节关于 loop unrolling 的知识。

总得来说，前面不算难，比较容易得分，但 Part C 想满分要花费大量的时间测试，有时改变一个跳转语句就能极大地影响性能。

还是要多动手，多看文档！

完整代码在 [这里](https://github.com/cwwang15/csapp-labs.git)，如果您能给我一个 star 就再好不过了（笑）。