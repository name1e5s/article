---
title: CS:APP Lab 4 解题报告
date: 2018-5-13 14:22:00
tags:
    - CSAPP
categories: CSAPP
---

CSAPP 的第四章《处理器体系结构》大多讲述的是 CPU 的实现，比较偏硬件。大约也是这个原因，很多高校在选用此教材讲述计算机系统导论课程时会直接跳过这一章。与之配套的 Lab 4，Architecture Lab 则更是无人问津。我趁着考完期中的空闲时间，花了近一天时间啃了下第四章，并顺便做了个下这个 Architecture Lab。

Architecture Lab 的主要目标是修改一个 `Y86-64` 汇编程序写就的函数 `ncopy` 以及我们使用的流水线 CPU 的 HCL 代码，使之在我们的 `Y86-64` 处理器上的效率尽量高（CPE(**C**locks **P**er **E**lement) 尽量小）。在这个任务之前，有两个小的任务来帮助我们熟悉相关的操作。

<!--more-->

### 实验前的准备

我们需要一个能用的 Linux 操作系统（笔者使用的是 elementaryOS ）。首先我们需要下载最新的实验材料：

```bash
$ wget http://csapp.cs.cmu.edu/im/labs/archlab.tar
```
之后对其解压：

```bash
$ tar xvf archlab-handout.tar
$ cd archlab-handout
$ tar xvf sim.tar
```
因为本套件依赖 Tcl/tk，我们需要安装这两个软件：

```bash
sudo apt install tcl tcl-dev tk tk-dev
```

因为 Makefile 里写的 tcl 的版本已经比较老了，我们需要修改一下 Makefile：

```bash
$ sed -i "s/tcl8.5/tcl8.6/g" Makefile
$ sed -i "s/CFLAGS=/CFLAGS=-DUSE_INTERP_RESULT /g" Makefile
```

之后即可正常编译套件了。执行编译：
```bash
$ cd sim
$ make clean; make
```

### Part A - 编写 Y86-64 汇编程序

在这里我们需要做的是将 `sim/misc` 下的 `example.c` 内含的三个函数改写成汇编版本的。直接以书上的图 4-7 为样本照猫画虎即可：

```asm
# sum_list.ys by name1e5s
# Execution begins at address 0

        .pos 0
        irmovq stack,%rsp
        call main
        halt

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

main:	irmovq ele1,%rdi
        call sum_list
	    ret

sum_list:
	    irmovq $0,%rax
	    jmp test
loop:	mrmovq 0(%rdi),%rsi
        addq %rsi,%rax
        mrmovq 8(%rdi),%rsi
        rrmovq %rsi,%rdi
test:   andq %rdi,%rdi
        jne loop
        ret

        .pos 0x100
stack:

```

写完第一个之后我们就可以使用如下指令进行测试辣：
```bash
$ ./yas sum.ys
$ ./yis sum.yo
Stopped in 29 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
Changes to registers:
%rax:	0x0000000000000000	0x0000000000000cba
%rsp:	0x0000000000000000	0x0000000000000100

Changes to memory:
0x00f0:	0x0000000000000000	0x000000000000005b
0x00f8:	0x0000000000000000	0x0000000000000013
```

输出结果在 `%rax` 内，直接进行比对即可。
与之类似的，我们可以写出剩下两个函数的 Y86-64 版本：

```asm
# rsum.ys by name1e5s
# Execution begins at address 0

	.pos 0
	irmovq stack, %rsp
	call main
	halt

# Sample linked list
	    .align 8
	    ele1:
	    .quad 0x00a
	    .quad e1e2
	    e1e2:
	    .quad 0x0b0
	    .quad e1e3
	    e1e3:
	    .quad 0xc00
	    .quad 0

main:	irmovq ele1, %rdi
	    call rsum_list
	    ret


rsum_list:
	    pushq %r12
	    irmovq $0, %rax
	    andq %rdi,%rdi
	    je re
	    mrmovq 0(%rdi), %r12
	    mrmovq 8(%rdi), %rdi
	    call rsum_list
	    addq %r12, %rax
re:	    popq %r12
	    ret

	    .pos 0x100
stack:
```

```asm
# cblock.ys by name1e5s
# Execution begins at address 0

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
	irmovq $3, %rdx
	call copy_block
	ret

copy_block:
	pushq %r12
	pushq %r13
	pushq %r14
	irmovq $1, %r13
	irmovq $8, %r14
	irmovq $0, %rax 
	jmp Tloop
loop:
	mrmovq 0(%rdi), %r12 
	addq %r14, %rdi
	rmmovq %r12, (%rsi)
	addq %r14, %rsi
	xorq %r12, %rax
	subq %r13, %rdx
Tloop:
	andq %rdx, %rdx
	jg loop
	popq %r14
	popq %r13
	popq %r12
	ret

	.pos 0x100
stack:
```

### Part B - 添加 iaddq 指令

直接按照 iaddq 的属性在 `sim/seq/seq-full.hcl` 中特定的位置添加 "IIADDQ" 即可，在此按下不表。

在添加完后，需要对其进行测试，执行如下指令：

```bash
$ make VERSION=full
$ (cd ../ptest; make SIM=../seq/ssim TFLAGS=-i)
```

如果没有问题，会返回如下结果：

```bash
 All 756 ISA Checks Succeed.
```

### Part C - 优化函数

这里我们要做的就是修改 `sim/pipe/pipe-full.hcl` 以及 'sim/pipe/ncopy.ys' 的内容。使我们的程序运行效率尽量高。在为 `pipe-full.hcl` 实现完 `iaddq` 之后。我们就可以分别使用如下指令测试我们的代码：
```bash
$ ./correctness.pl #结果是否正确
$ ./benchmark.pl #得出效率，分数越高结果越好
```

笔者的答案如下：

```asm
#/* $begin ncopy-ys */
##################################################################
# ncopy.ys - Copy a src block of len words to dst.
# Return the number of positive words (>0) contained in src.
#
# Include your name and ID here.
#
# Describe how and why you modified the baseline code.
#
##################################################################
# Do not modify this portion
# Function prologue.
# %rdi = src , %rsi = dst, %rdx = len
ncopy:

##################################################################
# You can modify this portion
	# Loop header
	xorq %rax,%rax		# count = 0
	iaddq $-4,%rdx		# length -= 4
	jl REM

Loop:
    mrmovq (%rdi), %r10	# read val from src...
	mrmovq 8(%rdi),%r11
	rmmovq %r10, (%rsi)	# ...and store it to dst
	andq %r10, %r10		# val <= 0?
	jle Npos		# if so, goto Npos:
	iaddq $1,%rax
Npos:
	rmmovq %r11,8(%rsi)
	andq %r11,%r11
	jle Npos2
	iaddq $1,%rax
Npos2:
	mrmovq 16(%rdi),%r10
	mrmovq 24(%rdi),%r11
	rmmovq %r10, 16(%rsi)
	andq %r10,%r10
	jle Npos3
	iaddq $1,%rax
Npos3:
	rmmovq %r11,24(%rsi)
	andq %r11,%r11
	jle nLoop
	iaddq $1,%rax
nLoop:
	iaddq $32,%rdi
	iaddq $32,%rsi
	iaddq $-4,%rdx
	jge Loop

REM:
	iaddq $3,%rdx
	jl Done
    mrmovq (%rdi), %r10
	mrmovq 8(%rdi),%r11
	rmmovq %r10, (%rsi)
	andq %r10,%r10
	jle RENPO
	iaddq $1,%rax
RENPO:
	iaddq $-1,%rdx
	jl Done
	rmmovq %r11,8(%rsi)
	andq %r11,%r11
	jle RENPO1
	iaddq $1,%rax
RENPO1:
	iaddq $-1,%rdx
	jl Done
    mrmovq 16(%rdi), %r10
	rmmovq %r10, 16(%rsi)
	andq %r10,%r10
	jle Done
	iaddq $1,%rax
##################################################################
# Do not modify the following section of code
# Function epilogue.
Done:
	ret
##################################################################
# Keep the following label at the end of your function
End:
#/* $end ncopy-ys */
```

笔者仅仅使用了两个最基础的优化策略：循环展开和提前为下一个步骤读取内存。这样就可以更好的利用 CPU 的流水线特性。笔者最终的得分是 48.6 分，中规中矩吧（但在网上有关新版 Architecture lab 的博文中也是最高的了（逃 ）。如果能为小的样例特殊优化下的话，结果应该能更好。

如果要继续优化的话，尝试修改下 CPU 的流水线策略也许是一个不错的选择。