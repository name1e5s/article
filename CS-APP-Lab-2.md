---
title: CS:APP Lab 2 解题报告
date: 2017-12-10 18:42:29
tags:
    - CSAPP
categories: CSAPP
---
CS:APP 的 Lab 2，也就是所谓的二进制炸弹，是一个很有趣的实验，相比 Lab 1 的单纯烧脑，Lab 2 更多的是根据反汇编获得的指令来推测推测程序的行为，最终给出符合要求的答案。本文中笔者使用的环境为 Ubuntu 17.10 Artful Aardvark。

<!--more-->

首先，使用objdump 指令获取反汇编文本，并保存到 bomb.S 中，之后打开 gdb ，载入文件准备调试。

## Phase_1

在 bomb.S 中可以看到函数 phase_1() 段的代码如下：

``` ASM
08048b33 <phase_1>:
 8048b33:	83 ec 14             	sub    $0x14,%esp
 8048b36:	68 8c a0 04 08       	push   $0x804a08c
 8048b3b:	ff 74 24 1c          	pushl  0x1c(%esp)
 8048b3f:	e8 52 05 00 00       	call   8049096 <strings_not_equal>
 8048b44:	83 c4 10             	add    $0x10,%esp
 8048b47:	85 c0                	test   %eax,%eax
 8048b49:	74 05                	je     8048b50 <phase_1+0x1d>
 8048b4b:	e8 3d 06 00 00       	call   804918d <explode_bomb>
 8048b50:	83 c4 0c             	add    $0xc,%esp
 8048b53:	c3                   	ret  
```

显然，此函数的作用就是将用户输入的字符串和保存在$0x804a08c 的字符串进行比较。故我们只需要取出此处的值，即为本段的答案。操作如下：
``` bash
(gdb) x/1s 0x804a08C
0x804a08c:  "Crikey! I have lost my mojo!"
```
因此炸弹一的答案为“Crikey! I have lost my mojo!”。

## Phase_2

我们先看代码：

``` ASM
08048b54 <phase_2>:
 8048b54:	56                   	push   %esi
 8048b55:	53                   	push   %ebx
 8048b56:	83 ec 2c             	sub    $0x2c,%esp
 8048b59:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048b5f:	89 44 24 24          	mov    %eax,0x24(%esp)
 8048b63:	31 c0                	xor    %eax,%eax
 8048b65:	8d 44 24 0c          	lea    0xc(%esp),%eax
 8048b69:	50                   	push   %eax
 8048b6a:	ff 74 24 3c          	pushl  0x3c(%esp)
 8048b6e:	e8 3f 06 00 00       	call   80491b2 <read_six_numbers>
 8048b73:	83 c4 10             	add    $0x10,%esp
 8048b76:	83 7c 24 04 00       	cmpl   $0x0,0x4(%esp)
 8048b7b:	75 07                	jne    8048b84 <phase_2+0x30>
 8048b7d:	83 7c 24 08 01       	cmpl   $0x1,0x8(%esp)
 8048b82:	74 05                	je     8048b89 <phase_2+0x35>
 8048b84:	e8 04 06 00 00       	call   804918d <explode_bomb>
 8048b89:	8d 5c 24 04          	lea    0x4(%esp),%ebx
 8048b8d:	8d 74 24 14          	lea    0x14(%esp),%esi
 8048b91:	8b 43 04             	mov    0x4(%ebx),%eax
 8048b94:	03 03                	add    (%ebx),%eax
 8048b96:	39 43 08             	cmp    %eax,0x8(%ebx)
 8048b99:	74 05                	je     8048ba0 <phase_2+0x4c>
 8048b9b:	e8 ed 05 00 00       	call   804918d <explode_bomb>
 8048ba0:	83 c3 04             	add    $0x4,%ebx
 8048ba3:	39 f3                	cmp    %esi,%ebx
 8048ba5:	75 ea                	jne    8048b91 <phase_2+0x3d>
 8048ba7:	8b 44 24 1c          	mov    0x1c(%esp),%eax
 8048bab:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048bb2:	74 05                	je     8048bb9 <phase_2+0x65>
 8048bb4:	e8 d7 fb ff ff       	call   8048790 <__stack_chk_fail@plt>
 8048bb9:	83 c4 24             	add    $0x24,%esp
 8048bbc:	5b                   	pop    %ebx
 8048bbd:	5e                   	pop    %esi
 8048bbe:	c3                   	ret    
```
根据代码

``` ASM
8048b6e:	e8 3f 06 00 00       	call   80491b2 <read_six_numbers>
```

我们可以推测出本次要求我们输入六个数字，将其根据下面的代码进行验证，如果验证失败则……BOOOOOOM！

再次阅读代码，发现有如下部分：

``` ASM
8048b76:	83 7c 24 04 00       	cmpl   $0x0,0x4(%esp)
8048b7b:	75 07                	jne    8048b84 <phase_2+0x30>
8048b7d:	83 7c 24 08 01       	cmpl   $0x1,0x8(%esp)
8048b82:	74 05                	je     8048b89 <phase_2+0x35>
8048b84:	e8 04 06 00 00       	call   804918d <explode_bomb>
```

这段代码的含义就是将这两个位置的数字与0和1进行比较，如果不相等则炸弹爆炸。因此我们猜测当此函数运行到 8048b76 时，我们输入的数字连续地存在于0x4(%esp) 到 0x24(%esp) 中。且第一个数字应为0，第二个应为1。现在，使用 gdb 调试。关键步骤如下：

``` ASM
(gdb) ni
0x08048b76 in phase_2 ()
(gdb) p *(int *)($esp + 0x4)
$1 = 0
(gdb) p *(int *)($esp + 0x8)
$2 = 1
(gdb) ni
0x08048b7b in phase_2 ()
(gdb) ni
0x08048b7d in phase_2 ()
```

完美！现在我们可以得出结论：我们输入的数字连续地存在于0x4(%esp) 到 0x24(%esp) 中，且第一个数字为0，第二个数字为1。

现在再看下面的代码：

``` ASM
8048b89:	8d 5c 24 04          	lea    0x4(%esp),%ebx
 8048b8d:	8d 74 24 14          	lea    0x14(%esp),%esi
 8048b91:	8b 43 04             	mov    0x4(%ebx),%eax
 8048b94:	03 03                	add    (%ebx),%eax
 8048b96:	39 43 08             	cmp    %eax,0x8(%ebx)
 8048b99:	74 05                	je     8048ba0 <phase_2+0x4c>
 8048b9b:	e8 ed 05 00 00       	call   804918d <explode_bomb>
 8048ba0:	83 c3 04             	add    $0x4,%ebx
 8048ba3:	39 f3                	cmp    %esi,%ebx
 8048ba5:	75 ea                	jne    8048b91 <phase_2+0x3d>
```
这段汇编是循环结构，略微难以理解，现在我们将其转写为等价的 C 语言代码：

``` C
/*
 * 假设用户输入的数据存在于数组 a[6] 中
 */
 
 for(int i = 2; i < 6 ;i++)
 {
     if(a[i] != (a[i - 1] + a[i - 2]))
         explode_bomb();
 }
```

因此，phase_2() 要求我们输入的是 Fibonacci
数列的第 0 ~ 5 项，即“0 1 1 2 3 5”。

经测试，答案正确。

## Phase_3

phase_3 的汇编代码如下：

``` ASM
08048bbf <phase_3>:
 8048bbf:	83 ec 28             	sub    $0x28,%esp
 8048bc2:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048bc8:	89 44 24 18          	mov    %eax,0x18(%esp)
 8048bcc:	31 c0                	xor    %eax,%eax
 8048bce:	8d 44 24 14          	lea    0x14(%esp),%eax
 8048bd2:	50                   	push   %eax
 8048bd3:	8d 44 24 13          	lea    0x13(%esp),%eax
 8048bd7:	50                   	push   %eax
 8048bd8:	8d 44 24 18          	lea    0x18(%esp),%eax
 8048bdc:	50                   	push   %eax
 8048bdd:	68 a9 a0 04 08       	push   $0x804a0a9
 8048be2:	ff 74 24 3c          	pushl  0x3c(%esp)
 8048be6:	e8 25 fc ff ff       	call   8048810 <__isoc99_sscanf@plt>
 8048beb:	83 c4 20             	add    $0x20,%esp
 8048bee:	83 f8 02             	cmp    $0x2,%eax
 8048bf1:	7f 05                	jg     8048bf8 <phase_3+0x39>
 8048bf3:	e8 95 05 00 00       	call   804918d <explode_bomb>
 8048bf8:	83 7c 24 04 07       	cmpl   $0x7,0x4(%esp)
 8048bfd:	0f 87 ef 00 00 00    	ja     8048cf2 <phase_3+0x133>
 8048c03:	8b 44 24 04          	mov    0x4(%esp),%eax
 8048c07:	ff 24 85 bc a0 04 08 	jmp    *0x804a0bc(,%eax,4)
 8048c0e:	b8 65 00 00 00       	mov    $0x65,%eax
 8048c13:	81 7c 24 08 20 02 00 	cmpl   $0x220,0x8(%esp)
 8048c1a:	00 
 8048c1b:	0f 84 db 00 00 00    	je     8048cfc <phase_3+0x13d>
 8048c21:	e8 67 05 00 00       	call   804918d <explode_bomb>
 8048c26:	b8 65 00 00 00       	mov    $0x65,%eax
 8048c2b:	e9 cc 00 00 00       	jmp    8048cfc <phase_3+0x13d>
 8048c30:	b8 68 00 00 00       	mov    $0x68,%eax
 8048c35:	83 7c 24 08 35       	cmpl   $0x35,0x8(%esp)
 8048c3a:	0f 84 bc 00 00 00    	je     8048cfc <phase_3+0x13d>
 8048c40:	e8 48 05 00 00       	call   804918d <explode_bomb>
 8048c45:	b8 68 00 00 00       	mov    $0x68,%eax
 8048c4a:	e9 ad 00 00 00       	jmp    8048cfc <phase_3+0x13d>
 8048c4f:	b8 72 00 00 00       	mov    $0x72,%eax
 8048c54:	83 7c 24 08 72       	cmpl   $0x72,0x8(%esp)
 8048c59:	0f 84 9d 00 00 00    	je     8048cfc <phase_3+0x13d>
 8048c5f:	e8 29 05 00 00       	call   804918d <explode_bomb>
 8048c64:	b8 72 00 00 00       	mov    $0x72,%eax
 8048c69:	e9 8e 00 00 00       	jmp    8048cfc <phase_3+0x13d>
 8048c6e:	b8 7a 00 00 00       	mov    $0x7a,%eax
 8048c73:	81 7c 24 08 09 03 00 	cmpl   $0x309,0x8(%esp)
 8048c7a:	00 
 8048c7b:	74 7f                	je     8048cfc <phase_3+0x13d>
 8048c7d:	e8 0b 05 00 00       	call   804918d <explode_bomb>
 8048c82:	b8 7a 00 00 00       	mov    $0x7a,%eax
 8048c87:	eb 73                	jmp    8048cfc <phase_3+0x13d>
 8048c89:	b8 72 00 00 00       	mov    $0x72,%eax
 8048c8e:	81 7c 24 08 1e 01 00 	cmpl   $0x11e,0x8(%esp)
 8048c95:	00 
 8048c96:	74 64                	je     8048cfc <phase_3+0x13d>
 8048c98:	e8 f0 04 00 00       	call   804918d <explode_bomb>
 8048c9d:	b8 72 00 00 00       	mov    $0x72,%eax
 8048ca2:	eb 58                	jmp    8048cfc <phase_3+0x13d>
 8048ca4:	b8 67 00 00 00       	mov    $0x67,%eax
 8048ca9:	81 7c 24 08 8a 02 00 	cmpl   $0x28a,0x8(%esp)
 8048cb0:	00 
 8048cb1:	74 49                	je     8048cfc <phase_3+0x13d>
 8048cb3:	e8 d5 04 00 00       	call   804918d <explode_bomb>
 8048cb8:	b8 67 00 00 00       	mov    $0x67,%eax
 8048cbd:	eb 3d                	jmp    8048cfc <phase_3+0x13d>
 8048cbf:	b8 72 00 00 00       	mov    $0x72,%eax
 8048cc4:	83 7c 24 08 65       	cmpl   $0x65,0x8(%esp)
 8048cc9:	74 31                	je     8048cfc <phase_3+0x13d>
 8048ccb:	e8 bd 04 00 00       	call   804918d <explode_bomb>
 8048cd0:	b8 72 00 00 00       	mov    $0x72,%eax
 8048cd5:	eb 25                	jmp    8048cfc <phase_3+0x13d>
 8048cd7:	b8 6f 00 00 00       	mov    $0x6f,%eax
 8048cdc:	81 7c 24 08 50 02 00 	cmpl   $0x250,0x8(%esp)
 8048ce3:	00 
 8048ce4:	74 16                	je     8048cfc <phase_3+0x13d>
 8048ce6:	e8 a2 04 00 00       	call   804918d <explode_bomb>
 8048ceb:	b8 6f 00 00 00       	mov    $0x6f,%eax
 8048cf0:	eb 0a                	jmp    8048cfc <phase_3+0x13d>
 8048cf2:	e8 96 04 00 00       	call   804918d <explode_bomb>
 8048cf7:	b8 68 00 00 00       	mov    $0x68,%eax
 8048cfc:	3a 44 24 03          	cmp    0x3(%esp),%al
 8048d00:	74 05                	je     8048d07 <phase_3+0x148>
 8048d02:	e8 86 04 00 00       	call   804918d <explode_bomb>
 8048d07:	8b 44 24 0c          	mov    0xc(%esp),%eax
 8048d0b:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048d12:	74 05                	je     8048d19 <phase_3+0x15a>
 8048d14:	e8 77 fa ff ff       	call   8048790 <__stack_chk_fail@plt>
 8048d19:	83 c4 1c             	add    $0x1c,%esp
 8048d1c:	c3                   	ret    
 ```

我们看到了熟悉的 sscanf() 函数，因此我们猜测其几行就是该函数的参数。现在我们查看 0x8048bdd push $0x804a0a9 中出现的常量 0x804a0a9 处的值。

``` bash
(gdb) x 0x804a0a9
0x804a0a9:  "%d %c %d"
```

稍有常识的人便可以看出，此字符串对应的就是我们应该输入的信息类型，即一个整数一个字符一个整数.

再往下看，可以发现如下代码：

``` ASM
8048beb:	83 c4 20             	add    $0x20,%esp
 8048bee:	83 f8 02             	cmp    $0x2,%eax
 8048bf1:	7f 05                	jg     8048bf8 <phase_3+0x39>
 8048bf3:	e8 95 05 00 00       	call   804918d <explode_bomb>
 8048bf8:	83 7c 24 04 07       	cmpl   $0x7,0x4(%esp)
 8048bfd:	0f 87 ef 00 00 00    	ja     8048cf2 <phase_3+0x133>
```

这段代码的作用就是判断用户输入个数是否大于 2 以及第一个数字是否小于 7，不符合以上两条件之一，则炸弹爆炸。

再向下，可以看到如下代码：

``` ASM
8048c07:	ff 24 85 bc a0 04 08 	jmp    *0x804a0bc(,%eax,4)
```

这一条的作用，就是按照我们输入的第一个数值进行跳转。这里，我们选择最简单的数值1进行操作。

``` bash
(gdb) p/x *(0x804a0bc + 4)
$1 = 0x8048c30
```

0x8048c30 处的代码如下：

``` ASM
8048c30:	b8 68 00 00 00       	mov    $0x68,%eax
 8048c35:	83 7c 24 08 35       	cmpl   $0x35,0x8(%esp)
 8048c3a:	0f 84 bc 00 00 00    	je     8048cfc <phase_3+0x13d>
 8048c40:	e8 48 05 00 00       	call   804918d <explode_bomb>
```

显然，将0x68 将转换为字符（’h’），即为需要输入的第三个字符，将 0x35 转化为十进制（’53’），即为最后一个数值。因此，答案为“1 h 53 ”。

## Phase_4

经过前三轮拆弹，我们对于汇编的阅读能力有了大幅度提升，因此这次我们直接查看关键代码。

``` ASM
8048d76:	83 ec 1c             	sub    $0x1c,%esp
 8048d79:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048d7f:	89 44 24 0c          	mov    %eax,0xc(%esp)
 8048d83:	31 c0                	xor    %eax,%eax
 8048d85:	8d 44 24 08          	lea    0x8(%esp),%eax
 8048d89:	50                   	push   %eax
 8048d8a:	8d 44 24 08          	lea    0x8(%esp),%eax
 8048d8e:	50                   	push   %eax
 8048d8f:	68 23 a2 04 08       	push   $0x804a223
 8048d94:	ff 74 24 2c          	pushl  0x2c(%esp)
 8048d98:	e8 73 fa ff ff       	call   8048810 <__isoc99_sscanf@plt>
```

此段代码的作用和上一个炸弹的作用差不多，即从输入中格式化读取数据。现在我们取出数据格式：

``` bash
(gdb) x/s 0x804a223
0x804a223:  "%d %d"
```

显然，我们需要输入两个整数。输入之后，来看下一段关键代码：

``` ASM
8048db1:	83 ec 04             	sub    $0x4,%esp
 8048db4:	6a 0e                	push   $0xe
 8048db6:	6a 00                	push   $0x0
 8048db8:	ff 74 24 10          	pushl  0x10(%esp)
 8048dbc:	e8 5c ff ff ff       	call   8048d1d <func4>
 8048dc1:	83 c4 10             	add    $0x10,%esp
 8048dc4:	83 f8 2d             	cmp    $0x2d,%eax
```

这段代码的含义就是将输入的第一个数字送入func4函数中，根据其返回值是否为0x2d来决定是否爆炸。现在来分析func4：

``` ASM
08048d1d <func4>:
 8048d1d:	56                   	push   %esi
 8048d1e:	53                   	push   %ebx
 8048d1f:	83 ec 04             	sub    $0x4,%esp
 8048d22:	8b 54 24 10          	mov    0x10(%esp),%edx
 8048d26:	8b 74 24 14          	mov    0x14(%esp),%esi
 8048d2a:	8b 4c 24 18          	mov    0x18(%esp),%ecx
 8048d2e:	89 c8                	mov    %ecx,%eax
 8048d30:	29 f0                	sub    %esi,%eax
 8048d32:	89 c3                	mov    %eax,%ebx
 8048d34:	c1 eb 1f             	shr    $0x1f,%ebx
 8048d37:	01 d8                	add    %ebx,%eax
 8048d39:	d1 f8                	sar    %eax
 8048d3b:	8d 1c 30             	lea    (%eax,%esi,1),%ebx
 8048d3e:	39 d3                	cmp    %edx,%ebx
 8048d40:	7e 15                	jle    8048d57 <func4+0x3a>
 8048d42:	83 ec 04             	sub    $0x4,%esp
 8048d45:	8d 43 ff             	lea    -0x1(%ebx),%eax
 8048d48:	50                   	push   %eax
 8048d49:	56                   	push   %esi
 8048d4a:	52                   	push   %edx
 8048d4b:	e8 cd ff ff ff       	call   8048d1d <func4>
 8048d50:	83 c4 10             	add    $0x10,%esp
 8048d53:	01 d8                	add    %ebx,%eax
 8048d55:	eb 19                	jmp    8048d70 <func4+0x53>
 8048d57:	89 d8                	mov    %ebx,%eax
 8048d59:	39 d3                	cmp    %edx,%ebx
 8048d5b:	7d 13                	jge    8048d70 <func4+0x53>
 8048d5d:	83 ec 04             	sub    $0x4,%esp
 8048d60:	51                   	push   %ecx
 8048d61:	8d 43 01             	lea    0x1(%ebx),%eax
 8048d64:	50                   	push   %eax
 8048d65:	52                   	push   %edx
 8048d66:	e8 b2 ff ff ff       	call   8048d1d <func4>
 8048d6b:	83 c4 10             	add    $0x10,%esp
 8048d6e:	01 d8                	add    %ebx,%eax
 8048d70:	83 c4 04             	add    $0x4,%esp
 8048d73:	5b                   	pop    %ebx
 8048d74:	5e                   	pop    %esi
```

这是一个递归函数，经过仔细推敲，我们就能知道这个函数的作用等同于如下 C 代码：

``` C
/*
 * 假设三个输入值分别为 a,b,c;
 * 第一次调用时候的数值分别为 用户输入的第一个数字，0，以及14
 */
int func4(int a,int b,int c)
{
        int average = (a + b)/2;
        if(a == average)
                return a;
        if(a < average)
                return average + func4(a,b,average-1);
        if(a > average)
                return average + func4(a,average+1,c);
}
```

经过穷举，得出输入的第一个数字应该为 14。

根据语句：

``` ASM
8048dc9:	83 7c 24 08 2d       	cmpl   $0x2d,0x8(%esp)
 8048dce:	74 05                	je     8048dd5 <phase_4+0x5f>
 8048dd0:	e8 b8 03 00 00       	call   804918d <explode_bomb>
 ```

立刻可知输入的第二个数据应该为 45。

## Phase_5

5.phase_5

第五题先看汇编代码，唔……这个代码让人绝望。因此引入新工具radare2来解决问题。

操作过程如下：

``` bash
name1e5s@ubuntu:~/bomb58$ r2 bomb
[0x080488e0]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[0x080488e0]> afl~phase
0x08048b33    3 33           sym.phase_1
0x08048b54   10 107          sym.phase_2
0x08048bbf    9 350  -> 122  sym.phase_3
0x08048d76    9 117          sym.phase_4
0x08048deb    9 128          sym.phase_5
0x08048e6b   24 230          sym.phase_6
0x08048fa2    5 92           sym.secret_phase
0x08049058    1 31           sym.invalid_phase
0x080492e6    8 161          sym.phase_defused
[0x080488e0]> s sym.phase_5
[0x08048deb]> pdf
/ (fcn) sym.phase_5 128
|   sym.phase_5 ();
|           ; var int local_bh @ esp+0xb
|           ; var int local_ch @ esp+0xc
|           ; var int local_11h @ esp+0x11
|           ; var int local_18h @ esp+0x18
|           ; arg int arg_2ch @ esp+0x2c
|              ; CALL XREF from 0x08048af9 (main)
|           0x08048deb      53             push ebx
|           0x08048dec      83ec24         sub esp, 0x24               ; '$'
|           0x08048def      8b5c242c       mov ebx, dword [arg_2ch]    ; [0x2c:4]=0x280009 ; ','
|           0x08048df3      65a114000000   mov eax, dword gs:[0x14]    ; [0x14:4]=1
|           0x08048df9      89442418       mov dword [local_18h], eax
|           0x08048dfd      31c0           xor eax, eax
|           0x08048dff      53             push ebx
|           0x08048e00      e872020000     call sym.string_length
|           0x08048e05      83c410         add esp, 0x10
|           0x08048e08      83f806         cmp eax, 6
|       ,=< 0x08048e0b      7405           je 0x8048e12
|       |   0x08048e0d      e87b030000     call sym.explode_bomb       ; long double expl(long double x)
|       |      ; JMP XREF from 0x08048e0b (sym.phase_5)
|       `-> 0x08048e12      b800000000     mov eax, 0
|              ; JMP XREF from 0x08048e2f (sym.phase_5)
|       .-> 0x08048e17      0fb61403       movzx edx, byte [ebx + eax]
|       |   0x08048e1b      83e20f         and edx, 0xf
|       |   0x08048e1e      0fb692dca004.  movzx edx, byte [edx + obj.array.3250] ; [0x804a0dc:1]=109 ; "maduiersnfotvbylWow! You've defused the secret stage!"
|       |   0x08048e25      88540405       mov byte [esp + eax + 5], dl
|       |   0x08048e29      83c001         add eax, 1
|       |   0x08048e2c      83f806         cmp eax, 6
|       `=< 0x08048e2f      75e6           jne 0x8048e17
|           0x08048e31      c644240b00     mov byte [local_bh], 0
|           0x08048e36      83ec08         sub esp, 8
|           0x08048e39      68b2a00408     push str.flames             ; 0x804a0b2 ; "flames"
|           0x08048e3e      8d442411       lea eax, dword [local_11h]  ; 0x11
|           0x08048e42      50             push eax
|           0x08048e43      e84e020000     call sym.strings_not_equal
|           0x08048e48      83c410         add esp, 0x10
|           0x08048e4b      85c0           test eax, eax
|       ,=< 0x08048e4d      7405           je 0x8048e54
|       |   0x08048e4f      e839030000     call sym.explode_bomb       ; long double expl(long double x)
|       |      ; JMP XREF from 0x08048e4d (sym.phase_5)
|       `-> 0x08048e54      8b44240c       mov eax, dword [local_ch]   ; [0xc:4]=0
|           0x08048e58      653305140000.  xor eax, dword gs:[0x14]
|       ,=< 0x08048e5f      7405           je 0x8048e66
|       |   0x08048e61      e82af9ffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
|       |      ; JMP XREF from 0x08048e5f (sym.phase_5)
|       `-> 0x08048e66      83c418         add esp, 0x18
|           0x08048e69      5b             pop ebx
\           0x08048e6a      c3             ret
```

可以看出，这段代码的作用就是将输入的六位字符串进行一顿复杂的操作后将其与0x804a0b2 处的字符串“flames”进行对比。因此，我们只需要找出偏移量，并匹配输出即可。

```bash
[0x08048deb]> px 16@obj.array.3250
- offset - 0 1 2 3 4 5 6
7 8 9 A B C D E F 0123456789ABCDEF
0x0804a0dc 6d61 6475 6965 7273 6e66 6f74 7662 796c maduiersnfotvbyl
[0x08048deb]> !rax2 -S flames
666c616d6573
```

将第二条指令输出的数字两位两位与上面的表格匹配，即可拿到偏移量，分别为9,0xF,1,0,5,7，编写如下 python 程序输出结果：

``` python
#!/usr/bin/python3

offsets = [9,15,1,0,5,7]
result = ""
for c in offsets:
    result += chr(c + 64)
print(result)
```

得到输出结果为 "IOA@EG"。

## Phase_6

终于来到第六题，我们先看汇编代码：

``` ASM
08048e6b <phase_6>:
 8048e6b:	56                   	push   %esi
 8048e6c:	53                   	push   %ebx
 8048e6d:	83 ec 4c             	sub    $0x4c,%esp
 8048e70:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 8048e76:	89 44 24 44          	mov    %eax,0x44(%esp)
 8048e7a:	31 c0                	xor    %eax,%eax
 8048e7c:	8d 44 24 14          	lea    0x14(%esp),%eax
 8048e80:	50                   	push   %eax
 8048e81:	ff 74 24 5c          	pushl  0x5c(%esp)
 8048e85:	e8 28 03 00 00       	call   80491b2 <read_six_numbers>
 8048e8a:	83 c4 10             	add    $0x10,%esp
 8048e8d:	be 00 00 00 00       	mov    $0x0,%esi
 8048e92:	8b 44 b4 0c          	mov    0xc(%esp,%esi,4),%eax
 8048e96:	83 e8 01             	sub    $0x1,%eax
 8048e99:	83 f8 05             	cmp    $0x5,%eax
 8048e9c:	76 05                	jbe    8048ea3 <phase_6+0x38>
 8048e9e:	e8 ea 02 00 00       	call   804918d <explode_bomb>
 8048ea3:	83 c6 01             	add    $0x1,%esi
 8048ea6:	83 fe 06             	cmp    $0x6,%esi
 8048ea9:	74 33                	je     8048ede <phase_6+0x73>
 8048eab:	89 f3                	mov    %esi,%ebx
 8048ead:	8b 44 9c 0c          	mov    0xc(%esp,%ebx,4),%eax
 8048eb1:	39 44 b4 08          	cmp    %eax,0x8(%esp,%esi,4)
 8048eb5:	75 05                	jne    8048ebc <phase_6+0x51>
 8048eb7:	e8 d1 02 00 00       	call   804918d <explode_bomb>
 8048ebc:	83 c3 01             	add    $0x1,%ebx
 8048ebf:	83 fb 05             	cmp    $0x5,%ebx
 8048ec2:	7e e9                	jle    8048ead <phase_6+0x42>
 8048ec4:	eb cc                	jmp    8048e92 <phase_6+0x27>
 8048ec6:	8b 52 08             	mov    0x8(%edx),%edx
 8048ec9:	83 c0 01             	add    $0x1,%eax
 8048ecc:	39 c8                	cmp    %ecx,%eax
 8048ece:	75 f6                	jne    8048ec6 <phase_6+0x5b>
 8048ed0:	89 54 b4 24          	mov    %edx,0x24(%esp,%esi,4)
 8048ed4:	83 c3 01             	add    $0x1,%ebx
 8048ed7:	83 fb 06             	cmp    $0x6,%ebx
 8048eda:	75 07                	jne    8048ee3 <phase_6+0x78>
 8048edc:	eb 1c                	jmp    8048efa <phase_6+0x8f>
 8048ede:	bb 00 00 00 00       	mov    $0x0,%ebx
 8048ee3:	89 de                	mov    %ebx,%esi
 8048ee5:	8b 4c 9c 0c          	mov    0xc(%esp,%ebx,4),%ecx
 8048ee9:	b8 01 00 00 00       	mov    $0x1,%eax
 8048eee:	ba 3c c1 04 08       	mov    $0x804c13c,%edx
 8048ef3:	83 f9 01             	cmp    $0x1,%ecx
 8048ef6:	7f ce                	jg     8048ec6 <phase_6+0x5b>
 8048ef8:	eb d6                	jmp    8048ed0 <phase_6+0x65>
 8048efa:	8b 5c 24 24          	mov    0x24(%esp),%ebx
 8048efe:	8d 44 24 24          	lea    0x24(%esp),%eax
 8048f02:	8d 74 24 38          	lea    0x38(%esp),%esi
 8048f06:	89 d9                	mov    %ebx,%ecx
 8048f08:	8b 50 04             	mov    0x4(%eax),%edx
 8048f0b:	89 51 08             	mov    %edx,0x8(%ecx)
 8048f0e:	83 c0 04             	add    $0x4,%eax
 8048f11:	89 d1                	mov    %edx,%ecx
 8048f13:	39 f0                	cmp    %esi,%eax
 8048f15:	75 f1                	jne    8048f08 <phase_6+0x9d>
 8048f17:	c7 42 08 00 00 00 00 	movl   $0x0,0x8(%edx)
 8048f1e:	be 05 00 00 00       	mov    $0x5,%esi
 8048f23:	8b 43 08             	mov    0x8(%ebx),%eax
 8048f26:	8b 00                	mov    (%eax),%eax
 8048f28:	39 03                	cmp    %eax,(%ebx)
 8048f2a:	7d 05                	jge    8048f31 <phase_6+0xc6>
 8048f2c:	e8 5c 02 00 00       	call   804918d <explode_bomb>
 8048f31:	8b 5b 08             	mov    0x8(%ebx),%ebx
 8048f34:	83 ee 01             	sub    $0x1,%esi
 8048f37:	75 ea                	jne    8048f23 <phase_6+0xb8>
 8048f39:	8b 44 24 3c          	mov    0x3c(%esp),%eax
 8048f3d:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 8048f44:	74 05                	je     8048f4b <phase_6+0xe0>
 8048f46:	e8 45 f8 ff ff       	call   8048790 <__stack_chk_fail@plt>
 8048f4b:	83 c4 44             	add    $0x44,%esp
 8048f4e:	5b                   	pop    %ebx
 8048f4f:	5e                   	pop    %esi
 8048f50:	c3                   	ret    
```

emmmmmmmmm……

![What The Fuck!!!!!!](https://coding.net/u/name1e5s/p/pic/git/raw/master/v2-6667f845856d4a9f67492fd4d640f89f_hd.jpg)

我们能够大致地看出，这段函数要求我们输入六个数字，而且不能重复且每个数字都得大于
0 小于等于 6，因此，我们的可能输入个数很少，仅仅 720 个，下面我们开始穷举：

``` python
FILE test.py :

#!/usr/bin/python3

def perm(items,n=None):
    if n == None:
        n = len(items)
    for i in range(len(items)):
        v = items[i:i+1]
        if(len(items) == 1):
            yield v
        else:
            rest = items[:i] + items[i+1:]
            for p in perm(rest,n-1):
                yield v + p
def prints(item):
    result = ""
    for a in item:
        result += a
        result += ' '
    print(result)

items = ['1','2','3','4','5','6']

for r in perm(items):
    prints(r)
```

``` bash
FILE test.sh:

#本程序中 ../Ans 为之前我们所求的前五个炸弹的解
#!/bin/bash

python3 test.py | while read line
do
        cat ../Ans > /tmp/test.txt
        echo $line >> /tmp/test.txt
        ./bomb /tmp/test.txt
        if [ "$?" -eq 0 ]; then
                echo The Answer Is :
                echo $line
                exit
        fi
done
```

执行程序之后获得输出如下：

``` bash
name1e5s@ubuntu:~/bomb58$ bash test.sh
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...

BOOM!!!
The bomb has blown up.

......

Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Congratulations! You've defused the bomb!
The Answer Is :
1 2 5 6 3 4
```
至此，六个炸弹悉数解开。

## 结束之后

真的结束了吗？

Naïve，并没有。

我们继续查看汇编码，可以看到在phase_6 之后还有如下神秘代码：

``` ASM
08048f51 <fun7>:
 8048f51:	53                   	push   %ebx
 8048f52:	83 ec 08             	sub    $0x8,%esp
 8048f55:	8b 54 24 10          	mov    0x10(%esp),%edx
 8048f59:	8b 4c 24 14          	mov    0x14(%esp),%ecx
 8048f5d:	85 d2                	test   %edx,%edx
 8048f5f:	74 37                	je     8048f98 <fun7+0x47>
 8048f61:	8b 1a                	mov    (%edx),%ebx
 8048f63:	39 cb                	cmp    %ecx,%ebx
 8048f65:	7e 13                	jle    8048f7a <fun7+0x29>
 8048f67:	83 ec 08             	sub    $0x8,%esp
 8048f6a:	51                   	push   %ecx
 8048f6b:	ff 72 04             	pushl  0x4(%edx)
 8048f6e:	e8 de ff ff ff       	call   8048f51 <fun7>
 8048f73:	83 c4 10             	add    $0x10,%esp
 8048f76:	01 c0                	add    %eax,%eax
 8048f78:	eb 23                	jmp    8048f9d <fun7+0x4c>
 8048f7a:	b8 00 00 00 00       	mov    $0x0,%eax
 8048f7f:	39 cb                	cmp    %ecx,%ebx
 8048f81:	74 1a                	je     8048f9d <fun7+0x4c>
 8048f83:	83 ec 08             	sub    $0x8,%esp
 8048f86:	51                   	push   %ecx
 8048f87:	ff 72 08             	pushl  0x8(%edx)
 8048f8a:	e8 c2 ff ff ff       	call   8048f51 <fun7>
 8048f8f:	83 c4 10             	add    $0x10,%esp
 8048f92:	8d 44 00 01          	lea    0x1(%eax,%eax,1),%eax
 8048f96:	eb 05                	jmp    8048f9d <fun7+0x4c>
 8048f98:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
 8048f9d:	83 c4 08             	add    $0x8,%esp
 8048fa0:	5b                   	pop    %ebx
 8048fa1:	c3                   	ret    

08048fa2 <secret_phase>:
 8048fa2:	53                   	push   %ebx
 8048fa3:	83 ec 08             	sub    $0x8,%esp
 8048fa6:	e8 42 02 00 00       	call   80491ed <read_line>
 8048fab:	83 ec 04             	sub    $0x4,%esp
 8048fae:	6a 0a                	push   $0xa
 8048fb0:	6a 00                	push   $0x0
 8048fb2:	50                   	push   %eax
 8048fb3:	e8 c8 f8 ff ff       	call   8048880 <strtol@plt>
 8048fb8:	89 c3                	mov    %eax,%ebx
 8048fba:	8d 40 ff             	lea    -0x1(%eax),%eax
 8048fbd:	83 c4 10             	add    $0x10,%esp
 8048fc0:	3d e8 03 00 00       	cmp    $0x3e8,%eax
 8048fc5:	76 05                	jbe    8048fcc <secret_phase+0x2a>
 8048fc7:	e8 c1 01 00 00       	call   804918d <explode_bomb>
 8048fcc:	83 ec 08             	sub    $0x8,%esp
 8048fcf:	53                   	push   %ebx
 8048fd0:	68 88 c0 04 08       	push   $0x804c088
 8048fd5:	e8 77 ff ff ff       	call   8048f51 <fun7>
 8048fda:	83 c4 10             	add    $0x10,%esp
 8048fdd:	83 f8 01             	cmp    $0x1,%eax
 8048fe0:	74 05                	je     8048fe7 <secret_phase+0x45>
 8048fe2:	e8 a6 01 00 00       	call   804918d <explode_bomb>
 8048fe7:	83 ec 0c             	sub    $0xc,%esp
 8048fea:	68 ec a0 04 08       	push   $0x804a0ec
 8048fef:	e8 cc f7 ff ff       	call   80487c0 <puts@plt>
 8048ff4:	e8 ed 02 00 00       	call   80492e6 <phase_defused>
 8048ff9:	83 c4 18             	add    $0x18,%esp
 8048ffc:	5b                   	pop    %ebx
 8048ffd:	c3                   	ret   
```

看来我们还没有完全破解 Dr.Evil 的炸弹，现在，我们查找这个函数的调用点，可以发现这个函数在<phase_defused>这里被调用，现在我们来分析<phase_defused>函数：

``` ASM
080492e6 <phase_defused>:
 80492e6:	83 ec 6c             	sub    $0x6c,%esp
 80492e9:	65 a1 14 00 00 00    	mov    %gs:0x14,%eax
 80492ef:	89 44 24 5c          	mov    %eax,0x5c(%esp)
 80492f3:	31 c0                	xor    %eax,%eax
 80492f5:	83 3d cc c3 04 08 06 	cmpl   $0x6,0x804c3cc
 80492fc:	75 73                	jne    8049371 <phase_defused+0x8b>
 80492fe:	83 ec 0c             	sub    $0xc,%esp
 8049301:	8d 44 24 18          	lea    0x18(%esp),%eax
 8049305:	50                   	push   %eax
 8049306:	8d 44 24 18          	lea    0x18(%esp),%eax
 804930a:	50                   	push   %eax
 804930b:	8d 44 24 18          	lea    0x18(%esp),%eax
 804930f:	50                   	push   %eax
 8049310:	68 7d a2 04 08       	push   $0x804a27d
 8049315:	68 d0 c4 04 08       	push   $0x804c4d0
 804931a:	e8 f1 f4 ff ff       	call   8048810 <__isoc99_sscanf@plt>
 804931f:	83 c4 20             	add    $0x20,%esp
 8049322:	83 f8 03             	cmp    $0x3,%eax
 8049325:	75 3a                	jne    8049361 <phase_defused+0x7b>
 8049327:	83 ec 08             	sub    $0x8,%esp
 804932a:	68 86 a2 04 08       	push   $0x804a286
 804932f:	8d 44 24 18          	lea    0x18(%esp),%eax
 8049333:	50                   	push   %eax
 8049334:	e8 5d fd ff ff       	call   8049096 <strings_not_equal>
 8049339:	83 c4 10             	add    $0x10,%esp
 804933c:	85 c0                	test   %eax,%eax
 804933e:	75 21                	jne    8049361 <phase_defused+0x7b>
 8049340:	83 ec 0c             	sub    $0xc,%esp
 8049343:	68 4c a1 04 08       	push   $0x804a14c
 8049348:	e8 73 f4 ff ff       	call   80487c0 <puts@plt>
 804934d:	c7 04 24 74 a1 04 08 	movl   $0x804a174,(%esp)
 8049354:	e8 67 f4 ff ff       	call   80487c0 <puts@plt>
 8049359:	e8 44 fc ff ff       	call   8048fa2 <secret_phase>
 804935e:	83 c4 10             	add    $0x10,%esp
 8049361:	83 ec 0c             	sub    $0xc,%esp
 8049364:	68 ac a1 04 08       	push   $0x804a1ac
 8049369:	e8 52 f4 ff ff       	call   80487c0 <puts@plt>
 804936e:	83 c4 10             	add    $0x10,%esp
 8049371:	8b 44 24 5c          	mov    0x5c(%esp),%eax
 8049375:	65 33 05 14 00 00 00 	xor    %gs:0x14,%eax
 804937c:	74 05                	je     8049383 <phase_defused+0x9d>
 804937e:	e8 0d f4 ff ff       	call   8048790 <__stack_chk_fail@plt>
 8049383:	83 c4 6c             	add    $0x6c,%esp
 8049386:	c3                   	ret    
```

可见关键点就在

``` ASM
 804932a:	68 86 a2 04 08       	push   $0x804a286
 804932f:	8d 44 24 18          	lea    0x18(%esp),%eax
 8049333:	50                   	push   %eax
 8049334:	e8 5d fd ff ff       	call   8049096 <strings_not_equal>
```

这四行中。

现在查看上文出现的常量地址中保存的数据：

``` bash
(gdb) x/s 0x804a286
0x804a286:	"DrEvil"
(gdb) x/s 0x804a27d
0x804a27d:	"%d %d %s"
```

可以推测出我们应该是在某个需要输入两个整数类型的地方多输入一个字符串"DrEvil"且前六个炸弹都拆掉才能执行<secret_phase>函数。回看之前六个炸弹，只有 phase_4 符合标准，因此我们把 "DrEvil" 加在第四个输入后面。

现在来看<secret_phase>函数，可以发现其将用户输入的转化为长整形之后与一参数 0x804c088 共同传入 fun7 内。现在我们来看查看 0x804c088 的内容：

``` bash
(gdb) x/120 0x804c088
0x804c088 <n1>: 36  134529172   134529184   8
0x804c098 <n21+4>:  134529220   134529196   50  134529208
0x804c0a8 <n22+8>:  134529232   22  134529304   134529280
0x804c0b8 <n33>:    45  134529244   134529316   6
0x804c0c8 <n31+4>:  134529256   134529292   107 134529268
0x804c0d8 <n34+8>:  134529328   40  0   0
0x804c0e8 <n41>:    1   0   0   99
0x804c0f8 <n47+4>:  0   0   35  0
0x804c108 <n44+8>:  0   7   0   0
0x804c118 <n43>:    20  0   0   47
0x804c128 <n46+4>:  0   0   1001    0
0x804c138 <n48+8>:  0   777 1   134529352
0x804c148 <node2>:  755 2   134529364   422
0x804c158 <node3+4>:    3   134529376   165 4
0x804c168 <node4+8>:    134529388   628 5   134529400
0x804c178 <node6>:  533 6   0   58
0x804c188:  0   0   0   0
0x804c198:  0   0   134521485   134521511
0x804c1a8 <host_table+8>:   134521537   0   0   0
0x804c1b8 <host_table+24>:  0   0   0   0
0x804c1c8 <host_table+40>:  0   0   0   0
0x804c1d8 <host_table+56>:  0   0   0   0
0x804c1e8 <host_table+72>:  0   0   0   0
0x804c1f8 <host_table+88>:  0   0   0   0
0x804c208 <host_table+104>: 0   0   0   0
0x804c218 <host_table+120>: 0   0   0   0
0x804c228 <host_table+136>: 0   0   0   0
0x804c238 <host_table+152>: 0   0   0   0
0x804c248 <host_table+168>: 0   0   0   0
0x804c258 <host_table+184>: 0   0   0   0
```

略去后面无用内容后不难看出改地址保存的是一颗二叉树，现在我们手动画出此二叉树：

![](https://coding.net/u/name1e5s/p/pic/git/raw/master/v2-e8ff0d563cf7fde690009b8a09301b1d_r.jpg)

有了此处的二叉树，上文的函数也不难分析出其等价的 C 语言代码：

``` C
typedef struct node {
    int data;
    node *left;
    node *right;
} node;

int fun7(node *pnode, int input){
    int ret;
    if(pNode == NULL)
        return ret;
    if(input < pnode -> data)
        ret = 2 * fun7(pnode -> left,input);
    else if(input > pnode -> data)
        ret = 2 * fun7(pnode -> right,input);
    else
        return 0;
}
```
而根据函数 <secret_phase> 中的语句

``` ASM
8048fdd: 83 f8 01  cmp $0x1,%eax
```

可知，<fun7> 的返回值应该为
1。不难推出我们的输入数值应该为 50。至此，二进制炸弹彻底破解完毕。正义的我再一次粉碎了 Dr.Evil 的阴谋。

全套答案如下：

```
Crikey! I have lost my mojo!
0 1 1 2 3 5
1 h 53
14 45 DrEvil
IOA@EG
1 2 5 6 3 4
50
```

至此，本 Lab 算是完成。

此炸弹还有一个彩蛋：当你试图使用 Ctrl-C 结束炸弹程序时候，将会输出如下内容：

``` bash
name1e5s@ubuntu:~/bomb58$ ./bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
^CSo you think you can stop the bomb with ctrl-c, do you?
Well...OK. :-)
```

充分地体现出了博士刀子嘴豆腐心的特点（误