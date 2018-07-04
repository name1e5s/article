---
title: 详解 C 语言链表（应用篇） -- 浅析 Linux kernel 中的 list.h 头文件（二）
date: 2018-03-17 21:00:00
tags:
    - C
categories: C
---

- [基础篇](https://kuso-kodo.github.io/2018/03/14/Linked-List-Basic/)

- [实践篇](https://kuso-kodo.github.io/2018/03/16/Linked-List-Midlevel/)

- [应用篇（一）](https://kuso-kodo.github.io/2018/03/17/Linked-List-Advanced-1/)


-----------------

注： 本文分析的为文章发布时最新的 Linux 内核源码。

-----------------

## 左移链表，即将链表的第一个节点移动到最后

```C
/**
 * list_rotate_left - rotate the list to the left
 * @head: the head of the list
 */
static inline void list_rotate_left(struct list_head *head)
{
	struct list_head *first;

	if (!list_empty(head)) {
		first = head->next;
		list_move_tail(first, head);
	}
}
```

处理完链表是空的的特殊情况之后，直接移动就好。

## 分割链表

```C
/**
 * list_cut_position - cut a list into two
 * @list: a new list to add all removed entries
 * @head: a list with entries
 * @entry: an entry within head, could be the head itself
 *	and if so we won't cut the list
 *
 * This helper moves the initial part of @head, up to and
 * including @entry, from @head to @list. You should
 * pass on @entry an element you know is on @head. @list
 * should be an empty list or a list you do not care about
 * losing its data.
 *
 */
static inline void list_cut_position(struct list_head *list,
		struct list_head *head, struct list_head *entry)
{
	if (list_empty(head))
		return;
	if (list_is_singular(head) &&
		(head->next != entry && head != entry))
		return;
	if (entry == head)
		INIT_LIST_HEAD(list);
	else
		__list_cut_position(list, head, entry);
}
```

函数的各个参数的含义如下：

list： 收留要被剪切的节点的链表

head： 要被剪切的链表

entry：剪切入口

代码在处理了一些特殊情况之后，使用一个单独的函数 `__list_cut_position` 来进行一般情况的处理。函数如下：

```C
static inline void __list_cut_position(struct list_head *list,
		struct list_head *head, struct list_head *entry)
{
	struct list_head *new_first = entry->next;
	list->next = head->next;
	list->next->prev = list;
	list->prev = entry;
	entry->next = list;
	head->next = new_first;
	new_first->prev = head;
}
```

总的来说，该函数的作用就是如下图：

![](https://coding.net/u/name1e5s/p/pic/git/raw/master/IMG_20180317_144320.jpg)

## 链表的合并

链表的合并共有如下四种形式。

```C
/**
 * list_splice - join two lists, this is designed for stacks
 * @list: the new list to add.
 * @head: the place to add it in the first list.
 */
static inline void list_splice(const struct list_head *list,
				struct list_head *head)
{
	if (!list_empty(list))
		__list_splice(list, head, head->next);
}

/**
 * list_splice_tail - join two lists, each list being a queue
 * @list: the new list to add.
 * @head: the place to add it in the first list.
 */
static inline void list_splice_tail(struct list_head *list,
				struct list_head *head)
{
	if (!list_empty(list))
		__list_splice(list, head->prev, head);
}

/**
 * list_splice_init - join two lists and reinitialise the emptied list.
 * @list: the new list to add.
 * @head: the place to add it in the first list.
 *
 * The list at @list is reinitialised
 */
static inline void list_splice_init(struct list_head *list,
				    struct list_head *head)
{
	if (!list_empty(list)) {
		__list_splice(list, head, head->next);
		INIT_LIST_HEAD(list);
	}
}

/**
 * list_splice_tail_init - join two lists and reinitialise the emptied list
 * @list: the new list to add.
 * @head: the place to add it in the first list.
 *
 * Each of the lists is a queue.
 * The list at @list is reinitialised
 */
static inline void list_splice_tail_init(struct list_head *list,
					 struct list_head *head)
{
	if (!list_empty(list)) {
		__list_splice(list, head->prev, head);
		INIT_LIST_HEAD(list);
	}
}
```

这四种形式无一例外，都是使用函数 `__list_splice` 进行实现，代码如下，写的极其简明直观。

```C
static inline void __list_splice(const struct list_head *list,
				 struct list_head *prev,
				 struct list_head *next)
{
	struct list_head *first = list->next;
	struct list_head *last = list->prev;

	first->prev = prev;
	prev->next = first;

	last->next = next;
	next->prev = last;
}
```

## 遍历

链表的各种遍历操作都是十分简单的，在 list.h 中便定义了很多简单的遍历宏。大多数宏都和如下宏类似：

```C
/**
 * list_for_each	-	iterate over a list
 * @pos:	the &struct list_head to use as a loop cursor.
 * @head:	the head for your list.
 */
#define list_for_each(pos, head) \
	for (pos = (head)->next; pos != (head); pos = pos->next)
```

这些操作过于简单，不再说明。

Linux 内核的链表遍历中的难点是根据链表节点的地址求出结构体的地址，最终使用结构体里面的数据。为此，内核里定义了一个使用 `container_of` 宏的 `list_entry` 宏。

```C
/**
 * list_entry - get the struct for this entry
 * @ptr:	the &struct list_head pointer.
 * @type:	the type of the struct this is embedded in.
 * @member:	the name of the list_head within the struct.
 */
#define list_entry(ptr, type, member) \
	container_of(ptr, type, member)
```

`container_of` 宏在 `linux.h` 中定义如下：

```C
// In [include/linux/kernel.h]
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:	the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:	the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({				\
	void *__mptr = (void *)(ptr);					\
	BUILD_BUG_ON_MSG(!__same_type(*(ptr), ((type *)0)->member) &&	\
			 !__same_type(*(ptr), void),			\
			 "pointer type mismatch in container_of()");	\
	((type *)(__mptr - offsetof(type, member))); })
```

上文中出现的 `offsetof` 宏定义如下：

```C
// In [include/linux/stddef.h]
#define offsetof(TYPE, MEMBER)	((size_t)&((TYPE *)0)->MEMBER)
```

因此，`container_of` 宏可以被简化为这样：

```C
const typeof(((type *)0)->member ) *__mptr = (ptr);
(type *)((char *)__mptr - ((size_t) &((type *)0)->member));
```

注意在上面我们使用了 `(char *)` 进行了强制类型转换以计算偏移量。

上面的 `typeof` 是 GCC 的拓展，求出某一量的类型，使用方法和 `sizeof` 类似。

以上です。