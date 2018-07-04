---
title: 【译】C Puzzles —— 有趣的 C 语言问题
date: 2018-01-29 07:24:01
tags:
    - C
    - 趣味问题
    - 翻译作品
categories: C
---
原文来自：[C PUZZLES, Some interesting C problems](http://www.gowrikumar.com/c/index.php)

本文文末有笔者的答案。

注意：原文中原作者和笔者使用的环境均为 linux + gcc，笔者的答案亦以此为准。

1 如下程序的期望输出是打印出数组里面的元素的内容，但是实际运行时并未如此。

``` C
#include<stdio.h>

  #define TOTAL_ELEMENTS (sizeof(array) / sizeof(array[0]))
  int array[] = {23,34,12,17,204,99,16};

  int main()
  {
      int d;

      for(d=-1;d <= (TOTAL_ELEMENTS-2);d++)
          printf("%d\n",array[d+1]);

      return 0;
  }
```

找出来是哪里的问题。

2 我认为下面这个程序十分完美。但是在编译时，我发现我犯了一个极其愚蠢的错误。你能发现吗？(在不使用编译器的情况下 :-)

``` C
#include<stdio.h>

void OS_Solaris_print()
{
        printf("Solaris - Sun Microsystems\n");
}

void OS_Windows_print()
{
        printf("Windows - Microsoft\n");

}
void OS_HP-UX_print()
{
        printf("HP-UX - Hewlett Packard\n");
}

int main()
{
        int num;
        printf("Enter the number (1-3):\n");
        scanf("%d",&num);
        switch(num)
        {
                case 1:
                        OS_Solaris_print();
                        break;
                case 2:
                        OS_Windows_print();
                        break;
                case 3:
                        OS_HP-UX_print();
                        break;
                default:
                        printf("Hmm! only 1-3 :-)\n");
                        break;
        }

        return 0;
}
```

3 以下程序的输出是什么？为什么？

``` C
enum {false,true};

int main()
{
        int i=1;
        do
        {
                printf("%d\n",i);
                i++;
                if(i < 15)
                        continue;
        }while(false);
        return 0;
}
```
4 如下程序看起来并没有正确输出"hello-out"(你可以试试)

``` C
  #include <stdio.h>
  #include <unistd.h>
  int main()
  {
          while(1)
          {
                  fprintf(stdout,"hello-out");
                  fprintf(stderr,"hello-err");
                  sleep(1);
          }
          return 0;
  }
```

为什么会这样？

5 乍一看这个程序可能会输出两条同样的字符串。

``` C
  #include <stdio.h>
  #define f(a,b) a##b
  #define g(a)   #a
  #define h(a) g(a)

  int main()
  {
          printf("%s\n",h(f(1,2)));
          printf("%s\n",g(f(1,2)));
          return 0;
  }
```

但是实际运行时，你得到的却是：

``` bash
bash$ ./a.out
12
f(1,2)
bash$
```

为什么？

6

```C
#include<stdio.h>
  int main()
  {
          int a=10;
          switch(a)
          {
                  case '1':
                      printf("ONE\n");
                      break;
                  case '2':
                      printf("TWO\n");
                      break;
                  defa1ut:
                      printf("NONE\n");
          }
          return 0;
  }
```

如果你以为如上程序的输出是 NONE ，那你可得好好看看了

7 如下 C 程序在 IA-64 下出现栈错误，但是在 IA-32 下却是好好的

``` C
int main()
  {
      int* p;
      p = (int*)malloc(sizeof(int));
      *p = 10;
      return 0;
  }
```

为什么会这样？

8 下面是一个计算数字的二进制里面有几个 1 的程序（才 14 行）

输入 输出

0 0(0000000)

5 2(0000101)

7 3(0000111)
``` C
int CountBits (unsigned int x )
  {
      static unsigned int mask[] = { 0x55555555,
          0x33333333,
          0x0F0F0F0F,
          0x00FF00FF,
          0x0000FFFF
          } ;

          int i ;
          int shift ; /* Number of positions to shift to right*/
          for ( i =0, shift =1; i < 5; i ++, shift *= 2)
                  x = (x & mask[i ])+ ( ( x >> shift) & mask[i]);
          return x;
  }
```

指出上面程序的运作原理。

9 如下程序的输出是什么，为什么？（如果你说结果是 "f is 1.0"）.那我建议你再检查检查。

```C
#include <stdio.h>

int main()
{
        float f=0.0f;
        int i;

        for(i=0;i<10;i++)
                f = f + 0.1f;

        if(f == 1.0f)
                printf("f is 1.0 \n");
        else
                printf("f is NOT 1.0\n");

        return 0;
}
```
10 我想下面这个程序是合法的（如果你看了逗号操作符号的用法的话）。但是这段程序还是有一个问题，你能发现吗？

``` C
#include <stdio.h>

int main()
{
        int a = 1,2;
        printf("a : %d\n",a);
        return 0;
}
```

11 如下程序的输出是什么？（这段程序合法吗？）

``` C
#include <stdio.h>
int main()
{
        int i=43;
        printf("%d\n",printf("%d",printf("%d",i)));
        return 0;
}
```

12 如下程序合法吗？如果合法，这段代码的意图是什么？为什么要这么写？

``` C
 void duff(register char *to, register char *from, register int count)
  {
      register int n=(count+7)/8;
      switch(count%8){
      case 0: do{ *to++ = *from++;
      case 7:  *to++ = *from++;
      case 6: *to++ = *from++;
      case 5: *to++ = *from++;
      case 4: *to++ = *from++;
      case 3: *to++ = *from++;
      case 2: *to++ = *from++;
      case 1: *to++ = *from++;
              }while( --n >0);
      }
  }
```

13 这是 CountBits (见第八题) 的另一种实现方法。验证其是否正确（怎么验证？？？）如果正确的话，指出这么操作的原因。

```C
 int CountBits(unsigned int x)
  {
      int count=0;
      while(x)
      {
          count++;
          x = x&(x-1);
      }
      return count;
  }
```

14 如下两个函数声明相同吗？

``` C
  int foobar(void);
  int foobar();
```

如下程序也许能够帮助你看出原因：（编译运行这两个程序，观察结果）

``` C
Program 1:
  #include <stdio.h>
  void foobar1(void)
  {
   printf("In foobar1\n");
  }

  void foobar2()
  {
   printf("In foobar2\n");
  }

  int main()
  {
     char ch = 'a';
     foobar1();
     foobar2(33, ch);
     return 0;
  }
Program 2:
  #include <stdio.h>
  void foobar1(void)
  {
   printf("In foobar1\n");
  }

  void foobar2()
  {
   printf("In foobar2\n");
  }

  int main()
  {
     char ch = 'a';
     foobar1(33, ch);
     foobar2();
     return 0;
  }
```

15 如下程序的输出是什么？为什么？

``` C
 #include <stdio.h>
  int main()
  {
   float a = 12.5;
   printf("%d\n", a);
   printf("%d\n", *(int *)&a);
   return 0;
  }
```
16 下面是分割成两个文件储存的 C 程序。当将这两个文件一同编译运行时，输出是什么？

``` C
File1.c
  int arr[80];
File2.c
  extern int *arr;
  int main()
  {
      arr[1] = 100;
      return 0;
  }
```
17 指出如下程序的输出。（不，不是 20）

``` C
  #include<stdio.h>
  int main()
  {
      int a=1;
      switch(a)
      {   int b=20;
          case 1: printf("b is %d\n",b);
                  break;
          default:printf("b is %d\n",b);
                  break;
      }
      return 0;
  }
```

18 如下程序的输出是什么？（也不是 40（如果 int 的大小是 4 的话））

```C

  #define SIZE 10
  void size(int arr[SIZE])
  {
          printf("size of array is:%d\n",sizeof(arr));
  }

  int main()
  {
          int arr[SIZE];
          size(arr);
          return 0;
  }
```

19 下面是一个简单的 C 程序，里面是一个调用 Error 函数显示错误信息的简单函数。你能发现 Error 函数的定义存在的隐患吗？

```C

  #include <stdlib.h>
  #include <stdio.h>
  void Error(char* s)
  {
      printf(s);
      return;
  }

  int main()
  {
      int *p;
      p = malloc(sizeof(int));
      if(p == NULL)
      {
          Error("Could not allocate the memory\n");
          Error("Quitting....\n");
          exit(1);
      }
      else
      {
          /*some stuff to use p*/
      }
      return 0;
  }
```

20 如下调用 scanf 的语句有何不同？（注意第二个调用处的空格。删掉再运行程序试试看）

```C
  #include <stdio.h>
  int main()
  {
      char c;
      scanf("%c",&c);
      printf("%c\n",c);

      scanf(" %c",&c);
      printf("%c\n",c);

      return 0;
  }
```

21 如下程序的潜在问题是什么？

``` C
  #include <stdio.h>
  int main()
  {
      char str[80];
      printf("Enter the string:");
      scanf("%s",str);
      printf("You entered:%s\n",str);

      return 0;
  }
```

22 如下程序的输出是什么？

```C
  #include <stdio.h>
  int main()
  {
      int i;
      i = 10;
      printf("i : %d\n",i);
      printf("sizeof(i++) is: %d\n",sizeof(i++));
      printf("i : %d\n",i);
      return 0;
  }
```

23 如下程序为什么会收到警告？(注意把正常的指针传入需要const指针不会爆警告)

```C
  #include <stdio.h>
  void foo(const char **p) { }
  int main(int argc, char **argv)
  {
          foo(argv);
          return 0;
  }
```
24 如下程序的输出是什么？

``` C
  #include <stdio.h>
  int main()
  {
          int i;
          i = 1,2,3;
          printf("i:%d\n",i);
          return 0;
  }

```
25 如下程序实现了逆波兰表达式计算器。程序里有一个或多个严重的 BUG ，找出它们！！！假设 getop 函数返回操作数，操作符，EOF 等等……

```C
  #include <stdio.h>
  #include <stdlib.h>

  #define MAX 80
  #define NUMBER '0'

  int getop(char[]);
  void push(double);
  double pop(void);
  int main()
  {
      int type;
      char s[MAX];

      while((type = getop(s)) != EOF)
      {
          switch(type)
          {
              case NUMBER:
                  push(atof(s));
                  break;
              case '+':
                  push(pop() + pop());
                  break;
              case '*':
                  push(pop() * pop());
                  break;
              case '-':
                  push(pop() - pop());
                  break;
              case '/':
                  push(pop() / pop());
                  break;
              /*   ... 
               *   ...    
               *   ... 
               */
          }
      }
  }
```

26 如下代码实现了 *nix 系统下的 banner 指令。指出代码的运作原理。

``` C
  #include<stdio.h>
  #include<ctype.h>

  char t[]={
      0,0,0,0,0,0,12,18,33,63,
      33,33,62,32,62,33,33,62,30,33,
      32,32,33,30,62,33,33,33,33,62,
      63,32,62,32,32,63,63,32,62,32,
      32,32,30,33,32,39,33,30,33,33,
      63,33,33,33,4,4,4,4,4,4,
      1,1,1,1,33,30,33,34,60,36,
      34,33,32,32,32,32,32,63,33,51,
      45,33,33,33,33,49,41,37,35,33,
      30,33,33,33,33,30,62,33,33,62,
      32,32,30,33,33,37,34,29,62,33,
      33,62,34,33,30,32,30,1,33,30,
      31,4,4,4,4,4,33,33,33,33,
      33,30,33,33,33,33,18,12,33,33,
      33,45,51,33,33,18,12,12,18,33,
      17,10,4,4,4,4,63,2,4,8,
      16,63
      };

  int main(int argc,char** argv)
  {

      int r,pr;
      for(r=0;r<6;++r)
          {
          char *p=argv[1];

          while(pr&&*p)
              {
              int o=(toupper(*p++)-'A')*6+6+r;
              o=(o<0||o>=sizeof(t))?0:o;
              for(pr=5;pr>=-1;--pr)
                  {
                  printf("%c",( ( (pr>=0) && (t[o]&(1<<pr)))?'#':' '));

                  }
              }
          printf("\n");
          }
      return 0;
  }
```

27 如下程序的输出是什么？

``` C
  #include <stdio.h>
  #include <stdlib.h>

  #define SIZEOF(arr) (sizeof(arr)/sizeof(arr[0]))

  #define PrintInt(expr) printf("%s:%d\n",#expr,(expr))
  int main()
  {
      /* The powers of 10 */
      int pot[] = {
          0001,
          0010,
          0100,
          1000
      };
      int i;

      for(i=0;i<SIZEOF(pot);i++)
          PrintInt(pot[i]);
      return 0;
  }
```

28 如下代码是欧几里得算法的实现。指出代码的运作逻辑，以及考虑当前试下是否还有可以改进的空间。

以及，scanf 函数的返回值是什么？

```C
  #include <stdio.h>
  int gcd(int u,int v)
  {
      int t;
      while(v > 0)
      {
          if(u > v)
          {
              t = u;
              u = v;
              v = t;
          }
          v = v-u;
      }
      return u;
  }

  int main()
  {
      int x,y;
      printf("Enter x y to find their gcd:");
      while(scanf("%d%d",&x, &y) != EOF)
      {
          if(x >0 && y>0)
              printf("%d %d %d\n",x,y,gcd(x,y));
                  printf("Enter x y to find their gcd:");
      }
      printf("\n");
      return 0;
  }
```

按照如上代码，实现一个计算四个数的最大公约数的代码。

29 如下代码的输出是什么？（绝对不是 10 ）

```C
  #include <stdio.h>
  #define PrintInt(expr) printf("%s : %d\n",#expr,(expr))
  int main()
  {
      int y = 100;
      int *p;
      p = malloc(sizeof(int));
      *p = 10;
      y = y/*p; /*dividing y by *p */;
      PrintInt(y);
      return 0;
  }
```

30 如下代码的作用是读入并输出日期。解释程序的行为。

``` C
  #include <stdio.h>
  int main()
  {
      int day,month,year;
      printf("Enter the date (dd-mm-yyyy) format including -'s:");
      scanf("%d-%d-%d",&day,&month,&year);
      printf("The date you have entered is %d-%d-%d\n",day,month,year);
      return 0;
  }
```

31 如下代码的作用是读入并输出一个整数。但是并不能正确工作。指出代码的错误。

```C
  #include <stdio.h>
  int main()
  {
      int n;
      printf("Enter a number:\n");
      scanf("%d\n",n);

      printf("You entered %d \n",n);
      return 0;
  }
```

32 如下代码意图使用二进制运算来执行乘 5 的操作。但是结果却不是如此。指出程序没有正确运行的原因。

```C
  #include <stdio.h>
  #define PrintInt(expr) printf("%s : %d\n",#expr,(expr))
  int FiveTimes(int a)
  {
      int t;
      t = a<<2 + a;
      return t;
  }

  int main()
  {
      int a = 1, b = 2,c = 3;
      PrintInt(FiveTimes(a));
      PrintInt(FiveTimes(b));
      PrintInt(FiveTimes(c));
      return 0;
  }
```

33 如下代码合法吗？

```C
  #include <stdio.h>
  #define PrintInt(expr) printf("%s : %d\n",#expr,(expr))
  int max(int x, int y)
  {
      (x > y) ? return x : return y;
  }

  int main()
  {
      int a = 10, b = 20;
      PrintInt(a);
      PrintInt(b);
      PrintInt(max(a,b));
  }
```

34 如下代码试图输出二十个减号，但是显然它没有正确运作。

```C
  #include <stdio.h>
  int main()
  {
      int i;
      int n = 20;
      for( i = 0; i < n; i-- )
          printf("-");
      return 0;
  }
```

直接修改过来是很简单的。为了好玩儿起见，你只能修改一个字符。有三种解决方法，试试看能否找全。

35 如下代码犯了什么错误？

```C
  #include <stdio.h>
  int main()
  {
      int* ptr1,ptr2;
      ptr1 = malloc(sizeof(int));
      ptr2 = ptr1;
      *ptr2 = 10;
      return 0;
  }
```
36 如下代码的输出是什么？

```C
 #include <stdio.h>
  int main()
  {
      int cnt = 5, a;

      do {
          a /= cnt;
      } while (cnt --);

      printf ("%d\n", a);
      return 0;
  }
```
37 如下代码的输出是什么？

```C
  #include <stdio.h>
  int main()
  {
      int i = 6;
      if( ((++i < 7) && ( i++/6)) || (++i <= 9))
          ;
      printf("%d\n",i);
      return 0;
  }
```

38 如下代码的 BUG 出现在哪里?

```C
  #include <stdlib.h>
  #include <stdio.h>
  #define SIZE 15 
  int main()
  {
      int *a, i;

      a = malloc(SIZE*sizeof(int));

      for (i=0; i<SIZE; i++)
          *(a + i) = i * i;
      for (i=0; i<SIZE; i++)
          printf("%d\n", *a++);
      free(a);
      return 0;
  }
```

39 如下 C 语言代码合法吗？输出是什么？

```C
  #include <stdio.h>
  int main()
  {
    int a=3, b = 5;

    printf(&a["Ya!Hello! how is this? %s\n"], &b["junk/super"]);
    printf(&a["WHAT%c%c%c  %c%c  %c !\n"], 1["this"],
       2["beauty"],0["tool"],0["is"],3["sensitive"],4["CCCCCC"]);
    return 0;
  }
```

40 如果输入是 Life is beautiful 的话，输出是什么？

```C
 #include <stdio.h>
  int main()
  {
      char dummy[80];
      printf("Enter a string:\n");
      scanf("%[^a]",dummy);
      printf("%s\n",dummy);
      return 0;
  }
```
41 注意：本题主要说明的是链接器而不是 C 语言。

我们有如下三个文件：

```C
a.c
---
int a;
b.c
---
int a = 10;
main.c
------
extern int a;
int main()
{
        printf("a = %d\n",a);
        return 0;
}
```

我们把它们一起编译一下，来看看结果：

``` bash
bash$ gcc a.c b.c main.c
bash$ ./a.out
a = 10
```

Hmm!!没有编译/链接错误！！为什么会这样？？

42 如下代码的作用是计算偏移量。指出这段宏的具体作用以及优点。

```C
#define offsetof(a,b) ((int)(&(((a*)(0))->b)))
```

43 如下宏是经典的使用异或交换两个变量的值的宏。指出这段代码的潜在问题。

```C
#define SWAP(a,b) ((a) ^= (b) ^= (a) ^= (b))
```

44 如下宏有何作用？

``` C
  #define DPRINTF(x) printf("%s:%d\n",#x,x)
```

45 假设你需要写一个 lAddOverFlow 函数。声明如下：

``` C
  int IAddOverFlow(int* result,int a,int b)
  {
      /* ... */
  }
```

结果存储在 result 指向的地址中，如果发生了溢出，结果即为 0。你需要怎么实现？（也就是，你怎么判断溢出？）

46 如下宏的作用是？

``` C
#define ROUNDUP(x,n) ((x+n-1)&(~(n-1)))
```

47 在大多数 C 语言书里，使用如下宏作为宏文件的例子。

``` C
  #define isupper(c) (((c) >= 'A') && ((c) <= 'Z'))
```

但是这么实现在按照如下文件的方法使用的话就会出现严重的问题（有何问题？）

```C
  char c;
  /* ... */
  if(isupper(c++))
  {
      /* ... */
  }
```

但是很多库文件还是将其实现为一个宏（无任何副作用）。看看你的操作系统上的 isupper() 是怎么实现的。

48 你应当知道省略号可以表示为函数的参数（printf 的声明是什么？）指出如下定义的错误之处。

```C
  int VarArguments(...)
  {
      /*....*/
      return 0;
  }
```

49 在不使用任何比较符号的情况下输出三个数字的最小值。

50 printf 的格式符 %n 的作用是什么？

51 只使用二进制运算符实现加法。

52 使用 printf 打印出 I can print %。

53 如下声明的区别是什么?

```C
  const char *p;
  char* const p;
```

54 memcpy 和 memmove 的区别是什么？

55 使用 printf 输出 double 以及 float 的格式描述符分别是什么？

56 写一个检测机器是大端序还是小端序的程序。

57 不使用分号，输出 Hello World!。

笔者答案：（前25题，后面的题目犯懒没有回答）

1 程序中的那个 for 循环不会被执行，因为 TOTAL_ELEMENTS 的类型是 size_t ，是一个无符号的类型。当操作中有有符号和无符号类型的量，且无符号的参数占存储空间的大小相比有符号的要大时，有符号量转换为无符号量。因此 -1 转换为 2^32-1 显然要比 TOTAL_ELEMENTS - 2 要大。进行一个强制类型转换即可使其正常工作。

2 在void OS_HP-UX_print() 里面有一个 ‘-’号，C语言的变量名不允许出现这个，将之删除或者替换为下划线可解。

3 1。continue 的作用是结束本次循环直接跳到循环条件处，因此只执行一次。

4 先是输出很多 hello-err 之后出现 hello-out。原因是 stdout 具有缓冲区，不刷新缓冲区域则不会输出，在 out 后面加 '\n' 即可解

5 本题说的是 C 中的宏中的 # 和 ## 的作用，详见：

    https://www.cnblogs.com/3me-linux/p/5825940.html

6 程序中没有 default 标签，出现的是 “defalut”,以后务必要小心拼写错误。

7 这是因为在 IA-64 里面，一个指针的长度是 64 位而不是一般情况下 int 的 32 位。在没有添加 stdlib.h 头文件时，malloc 函数的返回值会被假定为是 int 而不是指针。因此在 IA-64 下 malloc 返回的 64 位指针截断为 32 位，最后又被强制转换为 64 位的指针。这样显然会导致数据的丢失，最终在写入时造成 SEGFAULT 错误。在头文件处添加 stdlib.h 即可解决问题。

8 这个算法是使用分治法查出某数字的二进制有多少个 1。详见：

    https://en.wikipedia.org/wiki/Hamming_weight

9 输出结果：

```
f is NOT 1.0
```

浮点数精度问题。请见：

    https://stackoverflow.com/questions/9577179/c-floating-point-precision

浮点数之间的正确的比对方法是：

```C
#define EPSILON 0.000001f // Or whatever you like
inline int eq(float f1, float f2)
{
    return (fabs(f1 - f2) < EPSILON);
}
```

10 问题出在 int a = 1,2; 还是逗号的问题。详见：

    https://www.geeksforgeeks.org/comna-in-c-and-c/

11 合法，输出为 4321。一层一层展开即可。注意 printf 的返回值是输出的字符的个数。

12 所谓的 Duff's Device 。在 clang 的测试用例里还有词用例专门确保其能正确执行。详见：

    https://en.wikipedia.org/wiki/Duff%27s_device

13 还是经典的计算方法，如果一个整数不为0，那么这个整数至少有一位是1。如果把这个整数减去1，那么原来处在整数最右边的1就会变成0，原来在1后面的所有的0都会变成1。其余的所有位不受到影响。也就是说减1的结果是把从最右边一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做 AND 运算，从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000。也就是说，把一个整数减去1，再和原整数做 AND 运算，会把该整数最右边一个1变成0。一个整数的二进制有多少个1，就可以进行多少次这样的操作。

14 不一样，如果加了 void 函数就不能接收多余的参数，传参的话就会收到一个殡仪错误。但是没有 void 函数还可以加收参数，就是参数会被忽略掉。程序 1 是合法的。程序二无法通过编译。

15 输出如下：

```
0
1095237632
```

这个涉及到 printf 在输出与类型不符合的数字的处理方法以及数据在机器内部的存储方法。详见：

    https://stackoverflow.com/questions/4479758/float-to-int-unexpected-behaviour

16 结果喜人地简单：

```
Segmentation fault.
```

因为指针和数组不一样。如此写就会使 arr 的开头被视为指针，然后被解引用，最后导致 SEGFAULT。

17 有两种可能，在部分编译器上会给你一个警告，有的就会直接编译通过。输出是一堆乱码。为什么没有 “b 没有被定义”这种问题呢？ 这和编译器有关，通俗地说，编译器在读到 int b ... 这一句时就会预先设置好 b 变量，但是不会赋值。因此，b 被声明，但是没有初始化。

18 输出数字是 4 。对于函数，数组按照引用，也就是指针传递进去，因此结果就是 sizeof(int *)，大多数情况下就是 4。如果把这句话挪到 main 函数中，数字就是 40。

19 printf 的第一个形参需要为字符串常量，不是 char *。

20 第二个里面多余的空格可以跳过中间的回车等字符。

21 因为scanf 的实现问题这样只能读到第一个空格前面的内容，可以通过将之修改为

```C
scanf("%[^\n]",str);
```

来读入一行字符。

22 输出如下：

```
i : 10
sizeof(i++) is: 4
i : 10
```

sizeof 是编译时行为，结果在编译时就已经确定。

23 因为你试图将 char **传入需要 const char **的函数中。这两种类型间没有隐式类型转换。详见：

    https://stackoverflow.com/questions/7016098/implicit-conversion-from-char-to-const-char

24 输出为

```
i:1
```

逗号操作符，详见：

    https://en.wikipedia.org/wiki/Comma_operator

25 减法运算和除法运算对于操作数的顺序有要求。修改为如下代码即可:

```C
case '-':
    op2 = pop();
    push(pop() - op2);
    break;
case '/':
    op2 = pop();
    if (op2 != 0.0)
        push(pop() / op2);
    else
        printf("error: zero divisor\n");
    break;
```