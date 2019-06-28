---
title: 详解 C 语言链表（应用篇） -- 浅析 Linux kernel 中的 list.h 头文件（一）
date: 2018-03-17 09:00:00
tags:
    - C
categories: C
---

- [基础篇](https://kuso-kodo.github.io/2018/03/14/Linked-List-Basic/)

- [实践篇](https://kuso-kodo.github.io/2018/03/16/Linked-List-Midlevel/)

> 中国人有一句老话：“不入虎穴，焉得虎子。”这句话对于人们的实践是真理，对于认识论也是真理。离开实践的认识是不可能的。
>
>                                                  ---- 毛主席 《实践论》

计算机学科是一门实践性比较强的学科。这就意味着，搞计算机需要进行足够的实践。而回头看历史，计算机多年来的发展历程也是如此。人们从实践中发现理论，理论再指导新一轮的实践，循环往复，最终推动了技术的发展。

在前两篇文章中，我们讲述了基础的链表的实现方式。又留下了几个很有启发意义的习题，在本篇文章里面，我们来看简要地一下链表这一数据结构在 Linux 内核中的实际应用。

<!--more-->

-----------------

注： 本文分析的为文章发布时最新的 Linux 内核源码。

------------------

## Linux 内核中链表的声明和初始化

在 Linux 中，最常用的链表的定义如下：

```C
// In [include/linux/types.h]
struct list_head {
	struct list_head *next, *prev;
};
```

我们可以看到，Linux 内核中定义的是一个仅仅只有两个指针没有被数据位的双向循环链表结构体。因此，如果我们想要以此为基础定义一个能存储数据的链表结构的话，我们仅仅需要这样：

```C
struct useless_list {
    int                 data;
    struct list_head    list;
}
```

这样实现链表有如下优势：

1. 隐藏掉链表的指针属性，使其看着更加对人类友好。毕竟不是所有人都能像搞 Nginx 那帮老毛子一样轻轻松松的肝禁忌·四重指针的。

2. 链表的表示可以出现在任何位置。

3. 可以给链表任意命名。

4. 一个结构体可以属于多个链表，这是 Linux 内核的特性要求数据结构必须做到的。

看完链表的声明，我们来看链表的初始化宏。

```C
// In [include/linux/list.h] 以后若不特殊声明，全为此文件里的内容

#define LIST_HEAD_INIT(name) { &(name), &(name) }

#define LIST_HEAD(name) \
	struct list_head name = LIST_HEAD_INIT(name)
```

这里定义了初始化一个链表的宏，使得新建一个链表的流程被大大简化了。我们只需要一行。

```C
LIST_HEAD(fuc);
// 等价于
struct list_head fuc = LIST_HEAD_INIT(fuc);
```

就声明并初始化了一个首尾都指向自己的链表。在 list.h 中，除了初始化链表的宏，还有初始化的函数。定义如下：

```C
static inline void INIT_LIST_HEAD(struct list_head *list)
{
	WRITE_ONCE(list->next, list);
	list->prev = list;
}
```

不难看出，他们的作用完全相同。

## 增加/删除节点

### 增加节点

```C
/**
 * list_add - add a new entry
 * @new: new entry to be added
 * @head: list head to add it after
 *
 * Insert a new entry after the specified head.
 * This is good for implementing stacks.
 */
static inline void list_add(struct list_head *new, struct list_head *head)
{
	__list_add(new, head, head->next);
}


/**
 * list_add_tail - add a new entry
 * @new: new entry to be added
 * @head: list head to add it before
 *
 * Insert a new entry before the specified head.
 * This is useful for implementing queues.
 */
static inline void list_add_tail(struct list_head *new, struct list_head *head)
{
	__list_add(new, head->prev, head);
}
```

这是添加删除节点的函数实现，可以看到，这两个函数都是由一个函数 __list_add 负责具体实现的。现在我们看下这个函数。

```C
/*
 * Insert a new entry between two known consecutive entries.
 *
 * This is only for internal list manipulation where we know
 * the prev/next entries already!
 */
static inline void __list_add(struct list_head *new,
			      struct list_head *prev,
			      struct list_head *next)
{
	if (!__list_add_valid(new, prev, next))
		return;

	next->prev = new;
	new->next = next;
	new->prev = prev;
	WRITE_ONCE(prev->next, new);
}
```

这段代码做的就是在 prev 和 next 之间添加一个节点 new。因此，`__list_add(new, head, head->next)` 就是在 head 和 head->next 之间添加节点，就是所谓的头插法。而 `__list_add(new, head->prev, head)` 就是在 head->prev，即为最后一个链表的地址，和 head 之间添加一个节点，自然就是尾插法。

### 删除节点

```C
/*
 * Delete a list entry by making the prev/next entries
 * point to each other.
 *
 * This is only for internal list manipulation where we know
 * the prev/next entries already!
 */
static inline void __list_del(struct list_head * prev, struct list_head * next)
{
	next->prev = prev;
	WRITE_ONCE(prev->next, next);
}

/**
 * list_del - deletes entry from list.
 * @entry: the element to delete from the list.
 * Note: list_empty() on entry does not return true after this, the entry is
 * in an undefined state.
 */
static inline void __list_del_entry(struct list_head *entry)
{
	if (!__list_del_entry_valid(entry))
		return;

	__list_del(entry->prev, entry->next);
}

static inline void list_del(struct list_head *entry)
{
	__list_del_entry(entry);
	entry->next = LIST_POISON1;
	entry->prev = LIST_POISON2;
}
```

`__list_del_entry` 函数将 `entry` 的前一个节点和后一个节点之间建立关联。在 `__list_del_entry` 函数中出现的 `__list_del_entry_valid` 函数有着如下定义：

```C
// In [include/linux/list.h]
#ifdef CONFIG_DEBUG_LIST
extern bool __list_del_entry_valid(struct list_head *entry);
#else
static inline bool __list_del_entry_valid(struct list_head *entry)
{
	return true;
}
#endif

// In [lib/list_debug.c]
bool __list_del_entry_valid(struct list_head *entry)
{
	struct list_head *prev, *next;

	prev = entry->prev;
	next = entry->next;

	if (CHECK_DATA_CORRUPTION(next == LIST_POISON1,
			"list_del corruption, %p->next is LIST_POISON1 (%p)\n",
			entry, LIST_POISON1) ||
	    CHECK_DATA_CORRUPTION(prev == LIST_POISON2,
			"list_del corruption, %p->prev is LIST_POISON2 (%p)\n",
			entry, LIST_POISON2) ||
	    CHECK_DATA_CORRUPTION(prev->next != entry,
			"list_del corruption. prev->next should be %p, but was %p\n",
			entry, prev->next) ||
	    CHECK_DATA_CORRUPTION(next->prev != entry,
			"list_del corruption. next->prev should be %p, but was %p\n",
			entry, next->prev))
		return false;

	return true;

}
```

在非 DEBUG 模式下，该函数直接返回 `true`。在 DEBUG 模式下，该函数使用在 `lib/list_debug.c` 中的定义，该函数检查了四种链表不合法的情况，并根据情况输出对应的调试信息。在上文中出现的宏 ` LIST_POISON1` 和 ` LIST_POISON2` 的定义如下：

```C
// In [tools/include/linux/poison.h]

/*
 * These are non-NULL pointers that will result in page faults
 * under normal circumstances, used to verify that nobody uses
 * non-initialized list entries.
 */
#define LIST_POISON1  ((void *) 0x100 + POISON_POINTER_DELTA)
#define LIST_POISON2  ((void *) 0x200 + POISON_POINTER_DELTA)
```

注释说的很明确了，使用届个地址，可以造成内存分页错误。保证了这两个地址不会被访问到。

## 替换节点

```C
/**
 * list_replace - replace old entry by new one
 * @old : the element to be replaced
 * @new : the new element to insert
 *
 * If @old was empty, it will be overwritten.
 */
static inline void list_replace(struct list_head *old,
				struct list_head *new)
{
	new->next = old->next;
	new->next->prev = new;
	new->prev = old->prev;
	new->prev->next = new;
}
```
所谓替换节点就是修改和新旧节点相关的两个指针外加新节点自身的两个指针即可。

## 节点移动

所谓节点移动，就是将一个节点从旧的链表中删除之后，再加入到新的节点中。在 Linux 内核中有两个与之相关的函数，一个使用头插法，另一个使用尾插法。

```C
/**
 * list_move - delete from one list and add as another's head
 * @list: the entry to move
 * @head: the head that will precede our entry
 */
static inline void list_move(struct list_head *list, struct list_head *head)
{
	__list_del_entry(list);
	list_add(list, head);
}

/**
 * list_move_tail - delete from one list and add as another's tail
 * @list: the entry to move
 * @head: the head that will follow our entry
 */
static inline void list_move_tail(struct list_head *list,
				  struct list_head *head)
{
	__list_del_entry(list);
	list_add_tail(list, head);
}
```
两者操作相同，都是首先调用 `__list_del_entry(list)` ，将 list 的前一个结点和后一个结点建立联系，之后调用 `list_add(_tail)(list, head)` 将list添加到head的链表中。

## 一些检测操作

1. 检测是否为最后一个节点

```C
/**
 * list_empty - tests whether a list is empty
 * @head: the list to test.
 */
static inline int list_empty(const struct list_head *head)
{
	return READ_ONCE(head->next) == head;
}
```

因为 list.h 实现的是双向循环链表，所以可以判断节点是否为链表的最后一个就只需要判断其下一个节点是否为头节点即可。

2. 判断链表是否为空

```C
/**
 * list_empty_careful - tests whether a list is empty and not being modified
 * @head: the list to test
 *
 * Description:
 * tests whether a list is empty _and_ checks that no other CPU might be
 * in the process of modifying either member (next or prev)
 *
 * NOTE: using list_empty_careful() without synchronization
 * can only be safe if the only activity that can happen
 * to the list entry is list_del_init(). Eg. it cannot be used
 * if another CPU could re-list_add() it.
 */
static inline int list_empty_careful(const struct list_head *head)
{
	struct list_head *next = head->next;
	return (next == head) && (next == head->prev);
}
```

因为内核在运行时候可能有多个进程同时修改内存，判断链表是否为空需要判断他的前一个节点和后一个节点是否为空。但是注释也承认了这种判断方法的不安全性，要是保证安全操作，还需要加锁保护。

3. 判断链表是否只有一个节点

```C
/**
 * list_is_singular - tests whether a list has just one entry.
 * @head: the list to test.
 */
static inline int list_is_singular(const struct list_head *head)
{
	return !list_empty(head) && (head->next == head->prev);
}
```
很简单，不做出说明。