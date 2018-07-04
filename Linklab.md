---
title: Linklab 解题报告
date: 2018-01-11 07:24:01
tags:
    - CSAPP
categories: CSAPP
---

该 Lab 的附加材料会在文后提供下载。

Linklab 的主要作用就是帮助我们理解 ELF 文件格式的链接等操作，顺便复习一下之前学习的汇编知识。此次实验给我们提供了一个 main.o 以及五个phasex.o 文件。我们在每一阶段需要做的就是把 main.o 以及 phasex.o 文件链接起来，然后完成要求的输出。

## Phase 1

本阶段要求我们通过修改 phase1.o 中的 .rodata 字段，来实现修改输出为学号的目标。难度极小，为热身活动。

首先我们直接进行链接操作，查看结果：

``` bash
name1e5s@ubuntu:~/ll$ ./1
fZGFY ZYzVwwqe1HLlPI0uiWm24xVn1pFMpwVVOa5BIeIQW6ih7xbEeJ	kCz2qNa7XQo4	2DA3yYVeA3hfqF6AtTRvWK64r9KRROgH9Qmt7bNgrO
```

然后在 .data 字段查找这段乱码：

```
00000080: 5a66 5a47 4659 205a 597a 5677 7771 6531  ZfZGFY ZYzVwwqe1
  00000090: 484c 6c50 4930 7569 576d 3234 7856 6e31  HLlPI0uiWm24xVn1
  000000a0: 7046 4d70 7756 564f 6135 4249 6549 5157  pFMpwVVOa5BIeIQW
  000000b0: 3669 6837 7862 4565 4a09 6b43 7a32 714e  6ih7xbEeJ.kCz2qN
  000000c0: 6137 5851 6f34 0932 4441 3379 5956 6541  a7XQo4.2DA3yYVeA
  000000d0: 3368 6671 4636 4174 5452 7657 4b36 3472  3hfqF6AtTRvWK64r
  000000e0: 394b 5252 4f67 4839 516d 7437 624e 6772  9KRROgH9Qmt7bNgr
  000000f0: 4f00 0000 0000 0000 0047 4343 3a20 2855  O........GCC: (U
```
显然这段就是我们需要修改的部分，我们只需要将此修改为我们的学号即可。修改如下（假设笔者的学号为0000000000）：

```
00000080: 3030 3030 3030 3030 3030 3030 0071 6531  0000000000.Vwwqe1
  00000090: 484c 6c50 4930 7569 576d 3234 7856 6e31  HLlPI0uiWm24xVn1
```

之后链接运行，即可得到结果:

``` bash
name1e5s@ubuntu:~/ll$ ./1
0000000000
```

## Phase 2

本阶段，我们的目标仍是输出我们的学号，与上次不同的是，此次我们的学号已经预先存储在phase2.o 的 .rodata 字段，我们需要做的是补全 do_phase 函数，并修改相关内容，是指能够输出我们的学号。

首先进行反汇编：

```bash
name1e5s@ubuntu:~/ll$ objdump -d phase2.o 

phase2.o:     file format elf32-i386


Disassembly of section .text:

00000000 <ThbYPvDz>:
   0:	55                   	push   %ebp
   1:	89 e5                	mov    %esp,%ebp
   3:	83 ec 08             	sub    $0x8,%esp
   6:	83 ec 08             	sub    $0x8,%esp
   9:	68 00 00 00 00       	push   $0x0
   e:	ff 75 08             	pushl  0x8(%ebp)
  11:	e8 fc ff ff ff       	call   12 <ThbYPvDz+0x12>
  16:	83 c4 10             	add    $0x10,%esp
  19:	85 c0                	test   %eax,%eax
  1b:	75 10                	jne    2d <ThbYPvDz+0x2d>
  1d:	83 ec 0c             	sub    $0xc,%esp
  20:	ff 75 08             	pushl  0x8(%ebp)
  23:	e8 fc ff ff ff       	call   24 <ThbYPvDz+0x24>
  28:	83 c4 10             	add    $0x10,%esp
  2b:	eb 01                	jmp    2e <ThbYPvDz+0x2e>
  2d:	90                   	nop
  2e:	c9                   	leave  
  2f:	c3                   	ret    

00000030 <do_phase>:
  30:	55                   	push   %ebp
  31:	89 e5                	mov    %esp,%ebp
  33:	90                   	nop
  34:	90                   	nop
  35:	90                   	nop
  36:	90                   	nop
  37:	90                   	nop
  38:	90                   	nop
  39:	90                   	nop
  3a:	90                   	nop
  3b:	90                   	nop
  3c:	90                   	nop
  3d:	90                   	nop
  3e:	90                   	nop
  3f:	90                   	nop
  40:	90                   	nop
  41:	90                   	nop
  42:	90                   	nop
  43:	90                   	nop
  44:	90                   	nop
  45:	90                   	nop
  46:	90                   	nop
  47:	90                   	nop
  48:	90                   	nop
  49:	90                   	nop
  4a:	90                   	nop
  4b:	90                   	nop
  4c:	90                   	nop
  4d:	90                   	nop
  4e:	90                   	nop
  4f:	90                   	nop
  50:	90                   	nop
  51:	90                   	nop
  52:	90                   	nop
  53:	90                   	nop
  54:	5d                   	pop    %ebp
  55:	c3                   	ret    
```

我们要做的就是修改这些 nop 指令为我们自己的指令。上面那个乱码函数给了我们很好的示范，因此我们只需修改为如下即可：

``` bash
$ objdump -d phase2.o 

phase2.o:     file format elf32-i386


Disassembly of section .text:

00000000 <ThbYPvDz>:
   0:	55                   	push   %ebp
   1:	89 e5                	mov    %esp,%ebp
   3:	83 ec 08             	sub    $0x8,%esp
   6:	83 ec 08             	sub    $0x8,%esp
   9:	68 00 00 00 00       	push   $0x0
   e:	ff 75 08             	pushl  0x8(%ebp)
  11:	e8 fc ff ff ff       	call   12 <ThbYPvDz+0x12>
  16:	83 c4 10             	add    $0x10,%esp
  19:	85 c0                	test   %eax,%eax
  1b:	75 10                	jne    2d <ThbYPvDz+0x2d>
  1d:	83 ec 0c             	sub    $0xc,%esp
  20:	ff 75 08             	pushl  0x8(%ebp)
  23:	e8 fc ff ff ff       	call   24 <ThbYPvDz+0x24>
  28:	83 c4 10             	add    $0x10,%esp
  2b:	eb 01                	jmp    2e <ThbYPvDz+0x2e>
  2d:	90                   	nop
  2e:	c9                   	leave  
  2f:	c3                   	ret    

00000030 <do_phase>:
  30:	55                   	push   %ebp
  31:	89 e5                	mov    %esp,%ebp
  33:	83 ec 10             	sub    $0x10,%esp
  36:	68 00 00 00 00       	push   $0x0
  3b:	e8 fc ff ff ff       	call   3c <do_phase+0xc>
  40:	83 c4 10             	add    $0x10,%esp
  43:	90                   	nop
  44:	90                   	nop
  45:	90                   	nop
  46:	90                   	nop
  47:	90                   	nop
  48:	90                   	nop
  49:	90                   	nop
  4a:	90                   	nop
  4b:	90                   	nop
  4c:	90                   	nop
  4d:	90                   	nop
  4e:	90                   	nop
  4f:	90                   	nop
  50:	90                   	nop
  51:	90                   	nop
  52:	90                   	nop
  53:	90                   	nop
  54:	c9                   	leave  
  55:	c3                   	ret    
```

之后修改重定位规则，使得链接器重定位的是我们的代码，而非上面的乱码函数的代码。定位.rel.text字段以及其内容：

``` bash
name1e5s@ubuntu:~/ll$ readelf -r phase2.o

Relocation section '.rel.text' at offset 0x22c contains 3 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0000000a  00000501 R_386_32          00000000   .rodata
00000012  00000a02 R_386_PC32        00000000   strcmp
00000024  00000b02 R_386_PC32        00000000   puts

Relocation section '.rel.data' at offset 0x244 contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000000  00000c01 R_386_32          00000030   do_phase

Relocation section '.rel.eh_frame' at offset 0x24c contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000020  00000202 R_386_PC32        00000000   .text
00000040  00000202 R_386_PC32        00000000   .text
name1e5s@ubuntu:~/ll$ readelf -S phase2.o
There are 14 section headers, starting at offset 0x2c0:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        00000000 000034 000056 00  AX  0   0  1
  [ 2] .rel.text         REL             00000000 00022c 000018 08   I 12   1  4
  [ 3] .data             PROGBITS        00000000 00008c 000004 00  WA  0   0  4
  [ 4] .rel.data         REL             00000000 000244 000008 08   I 12   3  4
  [ 5] .bss              NOBITS          00000000 000090 000000 00  WA  0   0  1
  [ 6] .rodata           PROGBITS        00000000 000090 00000b 00   A  0   0  1
  [ 7] .comment          PROGBITS        00000000 00009b 00002e 01  MS  0   0  1
  [ 8] .note.GNU-stack   PROGBITS        00000000 0000c9 000000 00      0   0  1
  [ 9] .eh_frame         PROGBITS        00000000 0000cc 000058 00   A  0   0  4
  [10] .rel.eh_frame     REL             00000000 00024c 000010 08   I 12   9  4
  [11] .shstrtab         STRTAB          00000000 00025c 000063 00      0   0  1
  [12] .symtab           SYMTAB          00000000 000124 0000e0 10     13  10  4
  [13] .strtab           STRTAB          00000000 000204 000028 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)
```

找到目标后进行修改，因为我们只是用到了 puts 函数以及 .rodata 的位置，因此我们直接修改这两处的位置即可，修改后结果如下：

``` bash
name1e5s@ubuntu:~/linklab$ readelf -r phase2.o 

Relocation section '.rel.text' at offset 0x22c contains 3 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000037  00000501 R_386_32          00000000   .rodata
00000012  00000a02 R_386_PC32        00000000   strcmp
0000003c  00000b02 R_386_PC32        00000000   puts

Relocation section '.rel.data' at offset 0x244 contains 1 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000000  00000c01 R_386_32          00000030   do_phase

Relocation section '.rel.eh_frame' at offset 0x24c contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000020  00000202 R_386_PC32        00000000   .text
00000040  00000202 R_386_PC32        00000000   .text
name1e5s@ubuntu:~/linklab$ ./2
0000000000
```

可以看到，我们的代码已经被正确地重定位，并完美的执行。

## Phase_3

这个实验，我们还是得输出自己的学号，不过这次是利用 ELF 文件链接时候的强弱规则进行输出。我们先说些什么是强弱规则。

针对强弱符号的概念，链接器就会按如下规则处理与选择被多次定义的全局符号：

> · 规则1：不允许强符号被多次定义（即不同的目标文件中不能有同名的强符号）；如果有多个强符号定义，则链接器报符号重复定义错误。

>· 规则2：如果一个符号在某个目标文件中是强符号，在其他文件中都是弱符号，那么选择强符号。

>· 规则3：如果一个符号在所有目标文件中都是弱符号，那么选择其中占用空间最大的一个。比如目标文件A定义全局变量global为int型，占4个字节；目标文件B定义global为double型，占8个字节，那么目标文件A和B链接后，符号global占8个字节（尽量不要使用多个不同类型的弱符号，否则容易导致很难发现的程序错误）。

说完规则我们来看这个实验。利用 nm 指令，我们可以看到文件中的符号定义：

``` bash
name1e5s@ubuntu:~/linklab$ nm phase3.o 
00000000 T do_phase
00000100 C lOXXBDHjiI
00000000 D phase
         U putchar
         U __stack_chk_fail
```

在结合 readelf 的输出结果不难发现该文件中有一命名为 lOXXBDHjiI 的长度为256的字符串没有定义为强符号。既然如此，我们就先写如下 C 文件与之链接：

``` C
char lOXXBDHjiI[256] = "666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666";
```

结果如下：

``` bash
name1e5s@ubuntu:~/linklab$ ./3
6666666666
```

嗯，可以！非常6！

再使用 GDB 在 do_phase() 下断点；

``` bash
name1e5s@ubuntu:~/linklab$ gdb 3
GNU gdb (Ubuntu 8.0.1-0ubuntu1) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
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
Reading symbols from 3...(no debugging symbols found)...done.
(gdb) r
Starting program: /home/name1e5s/linklab/3 

[Inferior 1 (process 1833) exited normally]
(gdb) disas do_phase
Dump of assembler code for function do_phase:
   0x565555ed <+0>:	push   %ebp
   0x565555ee <+1>:	mov    %esp,%ebp
   0x565555f0 <+3>:	sub    $0x28,%esp
   0x565555f3 <+6>:	mov    %gs:0x14,%eax
   0x565555f9 <+12>:	mov    %eax,-0xc(%ebp)
   0x565555fc <+15>:	xor    %eax,%eax
   0x565555fe <+17>:	movl   $0x69716f67,-0x17(%ebp)
   0x56555605 <+24>:	movl   $0x776a6876,-0x13(%ebp)
   0x5655560c <+31>:	movw   $0x6378,-0xf(%ebp)
   0x56555612 <+37>:	movb   $0x0,-0xd(%ebp)
   0x56555616 <+41>:	movl   $0x0,-0x1c(%ebp)
   0x5655561d <+48>:	jmp    0x56555647 <do_phase+90>
   0x5655561f <+50>:	lea    -0x17(%ebp),%edx
   0x56555622 <+53>:	mov    -0x1c(%ebp),%eax
   0x56555625 <+56>:	add    %edx,%eax
   0x56555627 <+58>:	movzbl (%eax),%eax
   0x5655562a <+61>:	movzbl %al,%eax
   0x5655562d <+64>:	movzbl 0x2020(%eax),%eax
   0x56555634 <+71>:	movsbl %al,%eax
   0x56555637 <+74>:	sub    $0xc,%esp
   0x5655563a <+77>:	push   %eax
   0x5655563b <+78>:	call   0x5655563c <do_phase+79>
   0x56555640 <+83>:	add    $0x10,%esp
   0x56555643 <+86>:	addl   $0x1,-0x1c(%ebp)
   0x56555647 <+90>:	mov    -0x1c(%ebp),%eax
   0x5655564a <+93>:	cmp    $0x9,%eax
   0x5655564d <+96>:	jbe    0x5655561f <do_phase+50>
   0x5655564f <+98>:	sub    $0xc,%esp
   0x56555652 <+101>:	push   $0xa
   0x56555654 <+103>:	call   0x56555655 <do_phase+104>
   0x56555659 <+108>:	add    $0x10,%esp
   0x5655565c <+111>:	nop
   0x5655565d <+112>:	mov    -0xc(%ebp),%eax
   0x56555660 <+115>:	xor    %gs:0x14,%eax
   0x56555667 <+122>:	je     0x5655566e <do_phase+129>
   0x56555669 <+124>:	call   0x5655566a <do_phase+125>
   0x5655566e <+129>:	leave  
   0x5655566f <+130>:	ret    
End of assembler dump.
(gdb) b 0x56555634
Function "0x56555634" not defined.
Make breakpoint pending on future shared library load? (y or [n]) n
(gdb) b *0x56555634
Breakpoint 1 at 0x56555634
(gdb) r
Starting program: /home/name1e5s/linklab/3 

Breakpoint 1, 0x56555634 in do_phase ()
(gdb) i r
eax            0x0	0
ecx            0xffffd2b0	-11600
edx            0xffffd271	-11663
ebx            0x0	0
esp            0xffffd260	0xffffd260
ebp            0xffffd288	0xffffd288
esi            0x1	1
edi            0xf7fb5000	-134524928
eip            0x56555634	0x56555634 <do_phase+71>
eflags         0x286	[ PF SF IF ]
cs             0x23	35
ss             0x2b	43
ds             0x2b	43
es             0x2b	43
fs             0x0	0
gs             0x63	99
(gdb) disas
Dump of assembler code for function do_phase:
   0x565555ed <+0>:	push   %ebp
   0x565555ee <+1>:	mov    %esp,%ebp
   0x565555f0 <+3>:	sub    $0x28,%esp
   0x565555f3 <+6>:	mov    %gs:0x14,%eax
   0x565555f9 <+12>:	mov    %eax,-0xc(%ebp)
   0x565555fc <+15>:	xor    %eax,%eax
   0x565555fe <+17>:	movl   $0x69716f67,-0x17(%ebp)
   0x56555605 <+24>:	movl   $0x776a6876,-0x13(%ebp)
   0x5655560c <+31>:	movw   $0x6378,-0xf(%ebp)
   0x56555612 <+37>:	movb   $0x0,-0xd(%ebp)
   0x56555616 <+41>:	movl   $0x0,-0x1c(%ebp)
   0x5655561d <+48>:	jmp    0x56555647 <do_phase+90>
   0x5655561f <+50>:	lea    -0x17(%ebp),%edx
   0x56555622 <+53>:	mov    -0x1c(%ebp),%eax
   0x56555625 <+56>:	add    %edx,%eax
   0x56555627 <+58>:	movzbl (%eax),%eax
   0x5655562a <+61>:	movzbl %al,%eax
   0x5655562d <+64>:	movzbl 0x56557020(%eax),%eax
=> 0x56555634 <+71>:	movsbl %al,%eax
   0x56555637 <+74>:	sub    $0xc,%esp
   0x5655563a <+77>:	push   %eax
   0x5655563b <+78>:	call   0xf7e4ed50 <putchar>
   0x56555640 <+83>:	add    $0x10,%esp
   0x56555643 <+86>:	addl   $0x1,-0x1c(%ebp)
   0x56555647 <+90>:	mov    -0x1c(%ebp),%eax
   0x5655564a <+93>:	cmp    $0x9,%eax
   0x5655564d <+96>:	jbe    0x5655561f <do_phase+50>
   0x5655564f <+98>:	sub    $0xc,%esp
   0x56555652 <+101>:	push   $0xa
   0x56555654 <+103>:	call   0xf7e4ed50 <putchar>
   0x56555659 <+108>:	add    $0x10,%esp
   0x5655565c <+111>:	nop
   0x5655565d <+112>:	mov    -0xc(%ebp),%eax
   0x56555660 <+115>:	xor    %gs:0x14,%eax
   0x56555667 <+122>:	je     0x5655566e <do_phase+129>
   0x56555669 <+124>:	call   0xf7eeb860 <__stack_chk_fail>
   0x5655566e <+129>:	leave  
   0x5655566f <+130>:	ret    
End of assembler dump.
(gdb) x /s $eax
0x0:	<error: Cannot access memory at address 0x0>
(gdb) x /s $ebp-17
0xffffd277:	"jwxc"
(gdb) x /s $ebp - 0x17
0xffffd271:	"goqivhjwxc"
(gdb) x /s $ebp - 0x18
0xffffd270:	""
(gdb) x /s $ebp - 0x17
0xffffd271:	"goqivhjwxc"
(gdb) q
```

不难发现，这段代码其实等价于如下 C 代码：

``` C
void do_phase(void)
{
    char s[9] = "goqivhjwxc";
    for(int i = 0; i < 10 ; i++)
        printf("%c",s[i]);
    putchar('\n');
    return;
}
```

这就好办多了，我们只需要保证对应的字符为我们的学号就可以啦~

因此构造如下 C 文件：

``` C
char lOXXBDHjiI[256] = "666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666066600006666060666600066666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666";
```

与之链接，即可拿到结果：

``` bash
name1e5s@ubuntu:~/linklab$ ./3
0000000000
```

## Phase 4
这一关，考察的是对于switch 的掌握。但是，显然我们不需要理解 switch 的运行原理也可以解决问题。

我们先查看其反汇编码：

```bash
name1e5s@ubuntu:~/ll$ objdump -d phase4.o 

phase4.o:     file format elf32-i386


Disassembly of section .text:

00000000 <do_phase>:
   0:	55                   	push   %ebp
   1:	89 e5                	mov    %esp,%ebp
   3:	83 ec 28             	sub    $0x28,%esp
   6:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
   c:	89 45 f4             	mov    %eax,-0xc(%ebp)
   f:	31 c0                	xor    %eax,%eax
  11:	c7 45 e9 4f 58 59 44 	movl   $0x4459584f,-0x17(%ebp)
  18:	c7 45 ed 4b 50 56 47 	movl   $0x4756504b,-0x13(%ebp)
  1f:	66 c7 45 f1 46 5a    	movw   $0x5a46,-0xf(%ebp)
  25:	c6 45 f3 00          	movb   $0x0,-0xd(%ebp)
  29:	c7 45 e4 00 00 00 00 	movl   $0x0,-0x1c(%ebp)
  30:	e9 e2 00 00 00       	jmp    117 <do_phase+0x117>
  35:	8d 55 e9             	lea    -0x17(%ebp),%edx
  38:	8b 45 e4             	mov    -0x1c(%ebp),%eax
  3b:	01 d0                	add    %edx,%eax
  3d:	0f b6 00             	movzbl (%eax),%eax
  40:	88 45 e3             	mov    %al,-0x1d(%ebp)
  43:	0f be 45 e3          	movsbl -0x1d(%ebp),%eax
  47:	83 e8 41             	sub    $0x41,%eax
  4a:	83 f8 19             	cmp    $0x19,%eax
  4d:	0f 87 b0 00 00 00    	ja     103 <do_phase+0x103>
  53:	8b 04 85 00 00 00 00 	mov    0x0(,%eax,4),%eax
  5a:	ff e0                	jmp    *%eax
  5c:	c6 45 e3 33          	movb   $0x33,-0x1d(%ebp)
  60:	e9 9e 00 00 00       	jmp    103 <do_phase+0x103>
  65:	c6 45 e3 34          	movb   $0x34,-0x1d(%ebp)
  69:	e9 95 00 00 00       	jmp    103 <do_phase+0x103>
  6e:	c6 45 e3 74          	movb   $0x74,-0x1d(%ebp)
  72:	e9 8c 00 00 00       	jmp    103 <do_phase+0x103>
  77:	c6 45 e3 36          	movb   $0x36,-0x1d(%ebp)
  7b:	e9 83 00 00 00       	jmp    103 <do_phase+0x103>
  80:	c6 45 e3 6d          	movb   $0x6d,-0x1d(%ebp)
  84:	eb 7d                	jmp    103 <do_phase+0x103>
  86:	c6 45 e3 38          	movb   $0x38,-0x1d(%ebp)
  8a:	eb 77                	jmp    103 <do_phase+0x103>
  8c:	c6 45 e3 62          	movb   $0x62,-0x1d(%ebp)
  90:	eb 71                	jmp    103 <do_phase+0x103>
  92:	c6 45 e3 30          	movb   $0x30,-0x1d(%ebp)
  96:	eb 6b                	jmp    103 <do_phase+0x103>
  98:	c6 45 e3 79          	movb   $0x79,-0x1d(%ebp)
  9c:	eb 65                	jmp    103 <do_phase+0x103>
  9e:	c6 45 e3 3b          	movb   $0x3b,-0x1d(%ebp)
  a2:	eb 5f                	jmp    103 <do_phase+0x103>
  a4:	c6 45 e3 37          	movb   $0x37,-0x1d(%ebp)
  a8:	eb 59                	jmp    103 <do_phase+0x103>
  aa:	c6 45 e3 47          	movb   $0x47,-0x1d(%ebp)
  ae:	eb 53                	jmp    103 <do_phase+0x103>
  b0:	c6 45 e3 4d          	movb   $0x4d,-0x1d(%ebp)
  b4:	eb 4d                	jmp    103 <do_phase+0x103>
  b6:	c6 45 e3 5a          	movb   $0x5a,-0x1d(%ebp)
  ba:	eb 47                	jmp    103 <do_phase+0x103>
  bc:	c6 45 e3 3f          	movb   $0x3f,-0x1d(%ebp)
  c0:	eb 41                	jmp    103 <do_phase+0x103>
  c2:	c6 45 e3 39          	movb   $0x39,-0x1d(%ebp)
  c6:	eb 3b                	jmp    103 <do_phase+0x103>
  c8:	c6 45 e3 40          	movb   $0x40,-0x1d(%ebp)
  cc:	eb 35                	jmp    103 <do_phase+0x103>
  ce:	c6 45 e3 6b          	movb   $0x6b,-0x1d(%ebp)
  d2:	eb 2f                	jmp    103 <do_phase+0x103>
  d4:	c6 45 e3 70          	movb   $0x70,-0x1d(%ebp)
  d8:	eb 29                	jmp    103 <do_phase+0x103>
  da:	c6 45 e3 32          	movb   $0x32,-0x1d(%ebp)
  de:	eb 23                	jmp    103 <do_phase+0x103>
  e0:	c6 45 e3 31          	movb   $0x31,-0x1d(%ebp)
  e4:	eb 1d                	jmp    103 <do_phase+0x103>
  e6:	c6 45 e3 7c          	movb   $0x7c,-0x1d(%ebp)
  ea:	eb 17                	jmp    103 <do_phase+0x103>
  ec:	c6 45 e3 40          	movb   $0x40,-0x1d(%ebp)
  f0:	eb 11                	jmp    103 <do_phase+0x103>
  f2:	c6 45 e3 6e          	movb   $0x6e,-0x1d(%ebp)
  f6:	eb 0b                	jmp    103 <do_phase+0x103>
  f8:	c6 45 e3 35          	movb   $0x35,-0x1d(%ebp)
  fc:	eb 05                	jmp    103 <do_phase+0x103>
  fe:	c6 45 e3 5e          	movb   $0x5e,-0x1d(%ebp)
 102:	90                   	nop
 103:	0f be 45 e3          	movsbl -0x1d(%ebp),%eax
 107:	83 ec 0c             	sub    $0xc,%esp
 10a:	50                   	push   %eax
 10b:	e8 fc ff ff ff       	call   10c <do_phase+0x10c>
 110:	83 c4 10             	add    $0x10,%esp
 113:	83 45 e4 01          	addl   $0x1,-0x1c(%ebp)
 117:	8b 45 e4             	mov    -0x1c(%ebp),%eax
 11a:	83 f8 09             	cmp    $0x9,%eax
 11d:	0f 86 12 ff ff ff    	jbe    35 <do_phase+0x35>
 123:	83 ec 0c             	sub    $0xc,%esp
 126:	6a 0a                	push   $0xa
 128:	e8 fc ff ff ff       	call   129 <do_phase+0x129>
 12d:	83 c4 10             	add    $0x10,%esp
 130:	90                   	nop
 131:	8b 45 f4             	mov    -0xc(%ebp),%eax
 134:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 13b:	74 05                	je     142 <do_phase+0x142>
 13d:	e8 fc ff ff ff       	call   13e <do_phase+0x13e>
 142:	c9                   	leave  
 143:	c3                   	ret    
```

不难看出中间那一堆 jmp 等语句就是所谓的 switch 结构，以及 movb 后面的第一个操作数就是某些字符的 ANSCII 码，现在链接并运行程序：

``` bash
name1e5s@ubuntu:~/ll$ gcc -o 4 phase4.o main.o -m32
name1e5s@ubuntu:~/ll$ ./4
?n5679|b8^
```

并查看程序的二进制码：

``` bash
vim -b phase4.o
```

我们能在 phase4.o 里面看到一些被<e3><eb>包围的字符，这里就是之前看到的movb 后面的操作数的字符表示，我们只需把对应的字符改为我们的学号（对于笔者，就是把?n5679|b8^统统改为0）即可。

进行测试：

```bash
name1e5s@ubuntu:~/ll$ gcc -o 4 phase4.o main.o -m32
name1e5s@ubuntu:~/ll$ ./4
0000000000
```
### Phase 5

终于来到最后，也是最难办的一关。在进行实验前，我们先看下程序框架：

``` C
const int TRAN_ARRAY[] = {… …};
const char FDICT[] = FDICTDAT;
char BUF[] = MYID; 
char CODE = PHASE5_COOKIE;

int transform_code( int code, int mode )  {
    switch( TRAN_ARRAY [mode] & 0x00000007 )  {
        case 0:
            code = code & (~ TRAN_ARRAY[mode]);
            break;
        case 1:
            code = code ^ TRAN_ARRAY[mode];
            break;
        … …
    }
    return code;
}

void generate_code( int cookie )  {
    int i;
    CODE = cookie;
    for( i=0; i<sizeof(TRAN_ARRAY)/sizeof(int); i++ )
          CODE = transform_code( CODE, i );
}

int encode( char* str ) {
    int i, n = strlen(str);
    for( i=0; i<n; i++) {
        str[i] = (FDICT[str[i]] ^ CODE)& 0x7F;
        if( str[i]<0x20 || str[i]>0x7E ) str[i] = ' ';
    }
    return n;
}

void do_phase() {
    generate_code(PHASE5_COOKIE);
    encode(BUF);
    printf("%s\n", BUF);
}
```
这里的部分符号的重定位表被置为0了，我们需要做的就是将之修复，使得程序能够正常运行。操作很简单，在此按下不表。

实验材料下载地址：

https://pan.baidu.com/s/1dG64Pnr

密码：c1or

（学号已经置为 0000000000，不要妄图从中找到笔者的学号~）