---
title: 使用 C/C++ 模拟 defer 关键字
date: 2018-04-01 18:54:36
tags:
    - Go 语言
categories: Go
---

> 笔者在翻译完 [这篇文章](https://linux.cn/article-9268-1.html) 以及同系列的下一篇文章（尚未发布...）后，受到了 ESR 大神的鼓舞，遂决定在寒假学习一下 Go 语言。在学习 Go 语言的过程中,觉着这语言和之前学到的 C/C++ ，Scheme 相比有着无法比较的简洁感。笔者尤其喜欢 `defer` 这一关键字的设计。于是就在今天尝试使用 C/C++ 模拟了下 `defer` 关键字。

>                                                       ---- 某咸鱼的碎碎念

<!--more-->

## 关于 defer 的简介
`defer` 这一关键字可以说是 Go 语言的一大亮点。这一关键字通常用于资源的释放，会在函数返回之前进行调用。例如，当我们需要读取一个文件时，我们在 C/C++ 里面通常会如此操作：

```C
void inm() {
    FILE *fp;
    fp = fopen("114514.txt")

    if(!fp) {
        // open file failed
    }
    // blah blah
    if(ERROR OCCURED) {
        // handle
        fclose(fp);
        return;
    }

    // blah blah

    fclose(fp);
    return;
}
```

有了 `defer` 之后，我们在 Go 语言中即可进行如下操作：

```Go
func inm() {
    file,err := os.Open("114514.txt")
    if err != nil {
        // open file failed
    }

    defer file.Close()

    // blah blah

    return
}
```

这样我们就不必费心思考是否忘记关闭文件着一问题。只需处理我们真正需要处理的问题即可。同时代码的可读性也大幅提升。

## defer 的行为

粗略地讲，当我们在使用 `defer` 这一关键字时，我们的语句就会就会被加入到表中。在函数 `return` 前，这个表里的函数按照后进先出的顺序执行。因此，对于如下示例：

```Go
func zafechi3() { // ザフェチ3 
    defer fmt.Print(" 24 ")
    defer fmt.Print(" 歳 ")
    defer fmt.Print(" 学生です ")
    return
}
```
我们的输出就是 “ 学生です  歳  24 ”，而不是喜闻乐见的 “ 24  歳  学生です ”。

`defer` 的更多特性还有很多坑点都不在此说明。

## C++ 实现

因为 `defer` 关键字的特性和 C++ 的类的析构函数类似，我们即可很容易的模拟出 C++ 版本的 `defer`。如下是笔者的第一版 ·`defer`：

```C++
template <typename Function>
struct doDefer {
    Function f;
    doDefer(Function f) : f(f) {}
    ~doDefer() { f(); }
}

template <typename Function>
doDefer<Function> deferer(Function F) {
    return doDefer<Function>(f);
}

#define defer(expr) auto __defered = deferer([&](){expr;})
```

基本上实现了 `defer` 的功能，但是现在还无法在一个文件里多次调用 `defer`，不给力啊老师！

在经过了半个小时的查找之后，笔者发现了 `__COUNTER__` 宏，这个宏就相当于是一个计数器，在编译时，`__COUNTER__` 会被替换为它在源码中已经出现的次数。这个特性使得其很适合拿来区分多个重复的变量，示例如下：

```C++
#define   FUNC1(x,y)   x##y
#define   FUNC2(x,y)   FUNC1(x,y)
#define   FUNC(x)   FUNC2(x,__COUNTER__)

int FUNC(a)
int FUNC(a)
int FUNC(A)
```

在编译时，那三个 `int FUNC(a)` 会被依次展开为 `int a0`,`int a1`,`int a2`。利用这个特性，我们可以重写之前的 `defer`：

```C++
#define DEFER_1(x, y) x##y
#define DEFER_2(x, y) DEFER_1(x, y)
#define DEFER_0(x)    DEFER_2(x, __COUNTER__)
#define defer(expr)   auto DEFER_0(_defered_option) = deferer([&](){expr;})

template <typename Function>
struct doDefer {
    Function f;
    doDefer(Function f) : f(f) {}
    ~doDefer() { f(); }
}

template <typename Function>
doDefer<Function> deferer(Function F) {
    return doDefer<Function>(f);
}
```

这样，我们就能在 C++ 中愉快的使用 `defer` 辣~

### 与正版 defer 的区别

我们之前在说过，Go 语言的对于多个 `defer` 的处理是按照后进先出的顺序处理的。但是我们实现的 C++ 版本的 defer 显然不是如此，因为对于如下程序：

```C++
void inm() {
    defer(std::cout << " 24 ";)
    defer(std::cout <<" 歳 ";);
    defer(std::cout <<" 学生です ";);
    return;
}
```

我们的输出就是喜闻乐见的 “ 24  歳  学生です ”。不过，大致的功能还是相同的。

## C 实现

在 C，至少是标准 C 里面，我们无法优雅地实现 `defer` 这一关键字，但是由于 `gcc` 以及 `clang` 的拓展的存在，使得我们还是可以在短短几行内实现这一操作。

我们使用的拓展如下：

- [Common-Variable-Attributes](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#Common-Variable-Attributes)

- [C Block 闭包](http://clang.llvm.org/docs/BlockLanguageSpec.html)

其中 GCC 需要来自水果的补丁才能使用闭包功能，关于 Block 闭包，可以看[这里](https://www.jianshu.com/p/0d4b66a84448)。

有了这两个拓展，写出 C 语言版本的 `defer` 简直易如反掌：

```C
static inline void deferer(void (^*expr)(void)) { (*expr)(); }
#define DEFER_1(x,y) x##y
#define DEFER_2(x, y) DEFER_1(x, y)
#define DEFER_0(x)    DEFER_2(x, __COUNTER__)
#define defer __attribute__((cleanup(deferer))) void (^DEFER_0(_defered_option))(void) =
```

调用时候，只需：

```C
defer ^{ <expr> };
```

emmmmm.... 看着有点麻烦但也不是不能接受....

## 其他实现版本

写完着两份程序之后，笔者眉头一皱，觉得不简单。这个出现了近 10 年的关键字，肯定会有前辈实现过了。经过搜索，找出了如下几个，供读者欣赏：

-  [Ignacio Castaño](http://the-witness.net/news/2012/11/scopeexit-in-c11/)

- [gingerBill](http://www.gingerbill.org/article/defer-in-cpp.html)

- [Oded Lazar](https://oded.blog/2017/10/05/go-defer-in-cpp/)

- [Boost.ScopeExit](https://www.boost.org/doc/libs/1_66_0/libs/scope_exit/doc/html/index.html)

- [巨硬的 GSL](https://github.com/Microsoft/GSL/blob/ebe7ebfd855a95eb93783164ffb342dbd85cbc27/include/gsl/gsl_util#L85-L89)

```C++
// final_act allows you to ensure something gets run at the end of a scope
template <class F>
class final_act
{
public:
    explicit final_act(F f) noexcept : f_(std::move(f)), invoke_(true) {}

    final_act(final_act&& other) noexcept : f_(std::move(other.f_)), invoke_(other.invoke_)
    {
        other.invoke_ = false;
    }

    final_act(const final_act&) = delete;
    final_act& operator=(const final_act&) = delete;

    ~final_act() noexcept
    {
        if (invoke_) f_();
    }

private:
    F f_;
    bool invoke_;
};

// finally() - convenience function to generate a final_act
template <class F>
inline final_act<F> finally(const F& f) noexcept
{
    return final_act<F>(f);
}

template <class F>
inline final_act<F> finally(F&& f) noexcept
{
    return final_act<F>(std::forward<F>(f));
}
```

- pepper_chico

```C++
using defer = shared_ptr<void>;
```