---
title: malloc(0) 会发生什么
date: 2018-01-29 07:24:01
tags:
    - C
    - 趣味问题
categories: C
---



故事要从今天中午一位同学提到了这个问题开始......

![5b572595b42f1](https://i.loli.net/2018/07/24/5b572595b42f1.jpg)

这个问题看起来十分刁钻，不过稍有常识的人都知道，制定 C 标准的那帮语言律师也不是吃白饭的，对这种奇奇怪怪的问题一定会有定义。我们翻阅 [C17 标准](http://www.open-std.org/jtc1/sc22/wg14/www/abq/c17_updated_proposed_fdis.pdf) 草案 N2176，在 `7.22.3` 节里，有如下说法：

> The order and contiguity of storage allocated by successive calls to the aligned_alloc, calloc, malloc, and realloc functions is unspecified. The pointer returned if the allocation succeeds is suitably aligned so that it may be assigned to a pointer to any type of object with a fundamental alignment requirement and then used to access such an object or an array of such objects in the space allocated (until the space is explicitly deallocated). The lifetime of an allocated object extends from the allocation until the deallocation. Each such allocation shall yield a pointer to an object disjoint from any other object. The pointer returned points to the start (lowest byte address) of the allocated space. If the space cannot be allocated, a null pointer is returned. If the size of the space requested is zero, the behavior is implementation-defined: either a null pointer is returned to indicate an error, or the behavior is as if the size were some nonzero value, except that the returned pointer shall not be used to access an object.

在这里，标准委员会明确规定了：当 `malloc` 接到的参数为 0 时，其行为是由实现定义的（implementation-defined）。由实现定义的行为这个词就提醒我们，在实际编程时如果要考虑到程序在多个运行环境下进行运行时，不能对 `malloc` 返回的数值进行任何假设。换言之，没事儿不要吃饱了撑的在实际编程中写下 `malloc(0)` 这种天怒人怨的代码。

但是，这个无意义的问题吸引了我的兴趣。因此笔者开始查阅 `glibc` 的源代码，依此了解在 `glibc` 下，`mallloc(0)` 的行为。在 `glibc2.27/malloc/malloc.c` 中，有如下注释：

```c
/*
  malloc(size_t n)
  Returns a pointer to a newly allocated chunk of at least n bytes, or null
  if no space is available. Additionally, on failure, errno is
  set to ENOMEM on ANSI C systems.

  If n is zero, malloc returns a minumum-sized chunk. (The minimum
  size is 16 bytes on most 32bit systems, and 24 or 32 bytes on 64bit
  systems.)  On most systems, size_t is an unsigned type, so calls
  with negative arguments are interpreted as requests for huge amounts
  of space, which will often fail. The maximum supported value of n
  differs across systems, but is in all cases less than the maximum
  representable value of a size_t.
*/
```

注释已经说的很清楚了，当我们执行 `malloc(0)` 时，我们实际会拿到一个指向一小块内存的指针，这个指针指向的（分配给我们的）内存的大小是由机器决定的。西毒代码，可以发现，将读入的内存大小进行转换是由宏 `checked_request2size` 实现的。相关的宏定义如下：

```c
/* pad request bytes into a usable size -- internal version */

#define request2size(req)                                         \
  (((req) + SIZE_SZ + MALLOC_ALIGN_MASK < MINSIZE)  ?             \
   MINSIZE :                                                      \
   ((req) + SIZE_SZ + MALLOC_ALIGN_MASK) & ~MALLOC_ALIGN_MASK)

/* Same, except also perform an argument and result check.  First, we check
   that the padding done by request2size didn't result in an integer
   overflow.  Then we check (using REQUEST_OUT_OF_RANGE) that the resulting
   size isn't so large that a later alignment would lead to another integer
   overflow.  */
#define checked_request2size(req, sz) \
({				    \
  (sz) = request2size (req);	    \
  if (((sz) < (req))		    \
      || REQUEST_OUT_OF_RANGE (sz)) \
    {				    \
      __set_errno (ENOMEM);	    \
      return 0;			    \
    }				    \
})
```

也就是说，我们能申请到的数值最小为  `MINSIZE` ，这个 `MINSIZE` 的相关定义如下：

```c
/* The smallest possible chunk */
#define MIN_CHUNK_SIZE        (offsetof(struct malloc_chunk, fd_nextsize))

/* The smallest size we can malloc is an aligned minimal chunk */

#define MINSIZE  \
  (unsigned long)(((MIN_CHUNK_SIZE+MALLOC_ALIGN_MASK) & ~MALLOC_ALIGN_MASK))

/* The corresponding bit mask value.  */
#define MALLOC_ALIGN_MASK (MALLOC_ALIGNMENT - 1)

/* MALLOC_ALIGNMENT is the minimum alignment for malloc'ed chunks.  It
   must be a power of two at least 2 * SIZE_SZ, even on machines for
   which smaller alignments would suffice. It may be defined as larger
   than this though. Note however that code and data structures are
   optimized for the case of 8-byte alignment.  */
#define MALLOC_ALIGNMENT (2 * SIZE_SZ < __alignof__ (long double) \
			  ? __alignof__ (long double) : 2 * SIZE_SZ)

#ifndef INTERNAL_SIZE_T
# define INTERNAL_SIZE_T size_t
#endif

/* The corresponding word size.  */
#define SIZE_SZ (sizeof (INTERNAL_SIZE_T))

/*
  This struct declaration is misleading (but accurate and necessary).
  It declares a "view" into memory allowing access to necessary
  fields at known offsets from a given base. See explanation below.
*/

struct malloc_chunk {

  INTERNAL_SIZE_T      mchunk_prev_size;  /* Size of previous chunk (if free).  */
  INTERNAL_SIZE_T      mchunk_size;       /* Size in bytes, including overhead. */

  struct malloc_chunk* fd;         /* double links -- used only if free. */
  struct malloc_chunk* bk;

  /* Only used for large blocks: pointer to next larger size.  */
  struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};

// GCC 提供
/* Offset of member MEMBER in a struct of type TYPE. */
#define offsetof(TYPE, MEMBER) __builtin_offsetof (TYPE, MEMBER)
```

至此，我们就可以根据这些计算出使用 `glibc` 在我们的电脑上运行时 `malloc` 出的最小空间的大小了。计算完后，还可以根据 [**malloc_usable_size**](https://linux.die.net/man/3/malloc_usable_size) 判断自己的计算是否正确，样例代码如下：

```c
#include <stdio.h>
#include <malloc.h>

int main(void) {
    char *p = malloc(0);
    printf("Address: 0x%x.\nLength: %ld.\n",p,malloc_usable_size(p));
    return 0;
}
```

该样例在笔者电脑内输出的结果为 24。因此，我们知道了，在 `glibc` 下，执行 `malloc` 会得到一个指向分配给我们的大小为 `24` 字节的内存空间的指针。这只是在 `glibc` 下的结果，在其他 C 标准库实现内，可能你会得到一个空指针。因为标准中提到了，对于 `malloc(0)` 这种故意挑事的代码，实现时可以返回一个空指针作为回礼。


