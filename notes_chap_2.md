---
title: 数字逻辑 - 组合电路：小型设计 笔记
date: 2018-08-09 23:25:05
tags:
    - 数字逻辑设计与计算机组成
categories: 数字逻辑
mathjax: true
---

 这一章介绍的是设计小的组合电路时候的设计方法。

### 信号命名标准

对于一个信号，如果逻辑 1 表示有效，逻辑 0 表示无效，不活跃或者禁止的话，我们称此信号为高电平有效的，通常我们使用不带任何前缀或者后缀的名称来表示一个高电平有效的信号；如果逻辑 0 表示有效，逻辑 1 表示无效，不活跃或者禁止的话，该信号就是低电平有效的，我们一般使用带有<u>下划线</u>的名称来表示低电平有效信号。

对于单个信号，我们通常使用小写来表示，对于多个则一般用大写字母。比如 `x,X, _x , _X`分别定义了单个/多个 高电平/低电平信号。

<!--more-->

### 基础逻辑门

基础逻辑门共有如下七种，其符号以及真值表如下：

1. 与 (AND) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/and_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/and_truthtable.jpg)

2. 或 (OR) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/or_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/or_truthtable.jpg)

3. 非 (NOT) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/not_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/not_truthtable.jpg)

4. 与非 (NAND) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/nand_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/nand_truthtable.jpg)

5. 或非 (NOR) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/nor_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/nor_truthtable.jpg)

6. 异或 (XOR) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/xor_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/xor_truthtable.jpg)

7. 同或 (XNOR) 门

![符号](https://www.tutorialspoint.com/computer_logical_organization/images/xnor_logic.jpg)

![真值表](https://www.tutorialspoint.com/computer_logical_organization/images/xnor_truthtable.jpg)

*注释： 本段图片皆来自于 https://www.tutorialspoint.com/computer_logical_organization/logic_gates.htm

### 逻辑表达式

在逻辑表达式中,如果取遍了全部的输入值所产生的真值表和某一真值表是相同的，那么这个逻辑表达式就和其真值表是等价的。在逻辑表达式中我们分别使用 ·（或者不使用任何符号） , + 以及上划线分别表示与或非三种逻辑关系，比如一个逻辑表达式可以如下：

$ f(x) = \overline{xy} + xy$

#### 乘积的和（Sum of Product，SOP）表达式

当一个逻辑表达式定义了当输出为 1 时各个输入的值的时候，我们称其为乘积的和的表达式，这种表达式在形式上表示为几个乘积相加。例如：$f = \overline{xy} + xy$ 就是典型的乘积的和的表达式。

乘积的和表达式可以转化为使用与或非三个逻辑门构成的电路，比如 $f = \overline{xy} + xy$ 可以转化为：

![5b6ae3abb446e](https://i.loli.net/2018/08/08/5b6ae3abb446e.png)

##### 只用与非门实现 SOP 表达式

我们可以只使用与非门来构建一个 SOP 表达式的电路，该操作是基于如下两个基于德摩根律得来的转换得来的的：

$\overline {xy} = \bar x + \bar y$

$\overline {x + y} = \bar x \bar y$

例如，我们使用与非门构建$f = \overline{xy} + xy$ 的电路时的转换步骤如下：

1. 将非门替换为两个输入端连接到同一个信号的的与非门，因为 $\bar x = \overline {x \cdot x}$

2. 在 and1 和 and2 的后面分别添加两个非门，因为 $\bar {\bar x} = x$

3. 将与门-非门的组合使用与非门代替

4. 将非门-非门-或门的组合使用与非门代替，因为 $\overline {xy} = \bar x + \bar y$

基于这些步骤，我们不难得到表示$f = \overline{xy} + xy$的只使用与非门表示的电路。

#### 和的乘积（Product of Sum, POS）表达式

当一个逻辑表达式定义了当输出为 0 时各个输入的值的时候，我们称其为和的乘积的表达式，这种表达式在形式上表示为几个和式相乘。例如：$f = (x + \bar y)(\bar x + y)$ 就是典型的和的乘积的表达式。一旦 POS 表达式其中一个和式为 0 时，整个逻辑表达式的取值为 0。

##### SOP 与 POS 的转换

以下两个式子为 SOP 表达式与 POS 表达式之间的转换关系：

1. $ f 的 POS 表达式 = \overline {\bar f 的 SOP 表达式} $

2. $ f 的 SOP 表达式 = \overline {\bar f 的 POS 表达式} $

利用这两个式子以及来自德摩根的定律，我们可以轻松进行 SOP 表达式与 POS 表达式间的转换。例如，已知$f 的 SOP 表达式 = \bar x y + x \bar y$，求 $f$ 的 POS 表达式：

$f 的 POS 表达式 = \overline {\bar f 的 SOP 表达式} = \overline {\bar x y + x \bar y} = (\overline {\bar x y})(\overline {x \bar y}) = (\bar {\bar x} + \bar y)(\bar x + \bar {bar y}) = (x + \bar y)(\bar x + y)$

##### 对偶原理

将一个表达式中的与或符号互换后的表达式称为原表达式的对偶表达式。显然，对 SOP 的表达式取对偶形式之后对每一项取反后得到的就是 POS 表达式。证明不在赘述。

##### POS 表达式的实现

与 SOP 表达式类似，POS 表达式也可以使用与或非三种逻辑门实现，我们也可以仅仅使用或非门实现 POS 表达式，具体实现方法与使用与非门实现 SOP 表达式类似，不再说明。

概括的说，同一个函数的 SOP 表达式与 POS 表达式是等价的，两者产出相同的真实表，SOP 表达式用于与非门电路，POS 表达式用于或非门电路。

### 规范表达式（范式）

与离散中的概念类似，略去。

### 逻辑化简

目的：使用最少的逻辑门以及对于每个门使用最少的输入来生成输出。

#### 卡诺图

卡诺图是进行逻辑化简的一种方法，详见：http://www.eecg.toronto.edu/~ahouse/mirror/engi3861/kmaps.pdf

#### 逻辑化简算法

该算法又称为奎因-麦克拉斯基法，见： http://www.cs.columbia.edu/~cs6861/handouts/quine-mccluskey-handout.pdf

#### 化简软件

在数字逻辑与计算机组成一书中，作者向我们介绍了 Espresso 这一软件的应用。大致使用方法如下：

1. 使用如下语法编写输入文件：

   ```
   .i 4              # .i specifies the number of inputs 
   .o 3       # .o specifies the number of outputs 
   .ilb Q1 Q0 D N         # This line specifies the names of the inputs in order 
   .ob T1 T0 OPEN      # This line specifies the names of the outputs in order 
   0011 ---      #The first four digits (before the space) correspond 
   0101 110       # to the inputs, the three after the space correspond 
   0110 100       # to the outputs, both in order specified above. 
   0001 010 
   0010 100 
   0111 --- 
   1001 010 
   1010 010 
   1011 --- 
   1100 001 
   1101 001 
   1110 001 
   1111 --- 
   .e             # Signifies the end of the file.
   ```

2. 执行指令

   ```shell
   $> espresso temp.in
   ```

3. 即可看到输出，十分简洁：

   ```
    .i 4
    .o 3
    .ilb Q1 Q0 D N
    .ob T1 T0 OPEN
    .p 5
    0-1- 100
    101- 010
    11-- 001
    -0-1 010
    01-1 110
    .e
   ```

该软件的官方网址为：https://ptolemy.berkeley.edu/projects/embedded/pubs/downloads/espresso/index.htm ，但因为代码年代过于久远，我们无法在现代系统上直接进行编译，笔者会在近几天内修复好此份代码使之在现代操作系统上能够执行。

### 案例

#### 全加器

一个全加器有三个输入，通常记为 $a,b,c_{in}$，两个输出，记为 $s,c_{out}$。其中 $c_{in}$ 为进位输入，$c_{out}$为进位输出。其真值表如下：

**输入                 输出**

**A             B             CIN         COUT    S**

0              0              0              0              0

0              0              1              0              1

0              1              0              0              1

0              1              1              1              0

1              0              0              0              1

1              0              1              1              0

1              1              0              1              0

1              1              1              1              1

我们可以发现，全加器的作用可以视为是在求 $a,b,c_{in}$ 的和，并将个位输出到 $s$ ，进位数输出到 $c_{out}$ 里。不难发现，$s,c_{out} $ 的 SOP 表达式为：

$s(a,b,c_{in}) = \Sigma(1,2,4,7)$

$c_{out}(a,b,c_{in}) = \Sigma(3,5,6,7)$

利用化简技巧，不难求出最小 SOP 表达式为：

$s(a,b,c_{in}) = \bar a \bar bc_{in} + \bar ab \bar c_{in} + \bar abc_{in} + \overline {abc_{in}}$

$c_{out}(a,b,c_{in}) = ab + ac_{in} + bc_{in}$

或者，我们也可以使用异或操作进一步简化式子：

$s(a,b,c_{in}) = a \oplus b \oplus c_{in}$

$c_{out}(a,b,c_{in}) = ab + (a \oplus b)c_{in}$

但是，尽管使用异或门大大减少了门的个数，但是会增加信号传输的时间。

![Full Adder Circuit](http://www.circuitstoday.com/wp-content/uploads/2010/04/Full-Adder-Circuit.gif)

#### 多路选择器（MUX）

多路选择器一般有 $2^n $个输入信号以及 $n$ 个选择信号，其作用是根据选择信号选择多个输入信号中的一个。2-1 选择器的真值表如下：

| s   | x   | y   | r   |
| --- | --- | --- | --- |
| 1   | 1   | 1   | 1   |
| 1   | 1   | 0   | 0   |
| 1   | 0   | 1   | 1   |
| 1   | 0   | 0   | 0   |
| 0   | 1   | 1   | 1   |
| 0   | 1   | 0   | 1   |
| 0   | 0   | 1   | 0   |
| 0   | 0   | 0   | 0   |

显然，有

$r(s,x,y) = \Sigma (2,3,5,7)$

化简之后得到

$r(s,x,y) = sy + \bar sx$

对于更多扇入时，方法类似。

### 硬件描述语言（Verilog）

http://web.stanford.edu/class/ee183/tutorial/ee183_tutorial.pdf
