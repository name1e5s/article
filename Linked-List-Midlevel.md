---
title: 详解 C 语言链表（实践篇）
date: 2018-03-16 21:22:53
tags:
    - C
categories: C
---

在[基础篇](https://kuso-kodo.github.io/2018/03/14/Linked-List-Basic/)中，笔者介绍了一些链表的基本操作。在本文中，笔者整理了几个有关链表的问题供大家练习。练习的难度是递增的。前几道非常简单，后面就会愈加困难。答案会附在题目的后面。

注意：**仅仅用眼睛看答案不会提升自己的水平，只有自己亲手实现，才算真正的学到了知识。在看答案之前，无论你自己有没有成功实现，都请务必确保你真的认真思考了。**

优秀的程序员应该是能够将数据结构可视化的，因此他们能够很好的理解代码运行时内存的变化。链表的规整使得它很适合拿来做基本的可视化训练。在解决下面这些问题时，你可以尝试在纸上画出链表的结构进行模拟，最终使用代码将之表达起来。

## 约定

在下面的问题中，使用的定义如下：

```C
struct node {
    int             data;
    struct node*    next;
}

typedef struct node  node_t;
typedef struct node* nodeptr_t;
```

## 问题

按照要求实现如下函数。函数的声明已给出。

### 1. Count()

```C
/**
 * Count - 返回给定对象的出现次数
 *
 * @head:       查找的链表的头指针
 * @value:      需要查找的值
 */ 
int Count(nodeptr_t head, int value);
```

### 2. GetNth()

```C
/**
 * GetNth - 返回给定链表在给定索引处的值，如果超过链表的长度，应该返回一个错误代码。
 *
 * @head:       给定的链表
 * @index:      索引
 */
int GetNth(nodeptr_t head, int index);
```

 ### 3. DeleteList()

 ```C
 /**
  * DeleteList - 删除链表并置头指针为 NULL
  *
  * @headRef:      指向链表头指针的指针。
  */
void DeleteList(nodeptr_t* headRef);
```

### 4. SortedInsert()

```C
/**
 * SortedInsert - 像已排序的链表中插入新节点
 *
 * @headRef:    已排序的链表
 * @newNode:    新的节点
 */
void SortedInsert(nodeptr_t* headRef, nodeptr_t *newNode);
```

### 5. InsertSort()

```C
/**
 * InsertSort - 链表的插入排序
 *
 * @headRef:    链表的头指针的指针
 */
void InsertSort(nodeptr_t* headRef)
```

### 6. MergeSort()

```C
/**
 * InsertSort - 链表的插入排序
 *
 * @headRef:    链表的头指针的指针
 */
void MergetSort(nodeptr_t* headRef)
```
### 7. Reverse()

```C
/**
 * InsertSort - 链表的插入排序
 *
 * @headRef:    链表的头指针的指针
 */
void Reverse(nodeptr_t* headRef)
/**
 * 楼上的递归版本
 */
void ReverseRecursive(nodeptr_t* headRef)
```

提示：

![](https://coding.net/u/name1e5s/p/pic/git/raw/master/Reverse.png)

其中红色为最初的链表，绿色为最终的结果。

## 答案

### 1. Count()

本函数的实现极其简单。

```C
int Count(nodeptr_t head, int value) {
    nodeptr_t current;
    int count = 0;
    for (current = head; current != NULL;  current = current->next) {
        if(current->data == searchFor)
            count++;
        }
    return count;
}
```

### 2. GetNth()

```C
int GetNth(nodeptr_t head, int index) {
    nodeptr_t current = head;
    int count = 0;
    while (current != NULL) {
        if(count == index)
            return(current->data);
        count++;
        current = current->next;
        }
    assert(0); // 报错 
}
```

### 3. DeleteList()

```C
void DeleteList(nodeptr_t* headRef) {
    nodeptr_t current = *headRef;
    while (current != NULL) {
        next = current->next;
        free(current);
        current = next;
    }
    *headRef = NULL;
}
```

### 4. SortedInsert()

```C
void SortedInsert(nodeptr_t* headRef, nodeptr_t *newNode) {
    // 处理特殊情况
    if (*(headRef) == NULL || (*headRef)->data >=   newNode->data) {
        newNode->next = *headRef;
        *headRef = newNode;
    }
    else {
        nodeptr_t current = *headRef;
        while (current->next!=NULL && current->next->data<newNode->data) {
            current = current->next;
        }
        newNode->next = current->next;
        current->next = newNode;
    }
}
```

### 5. InsertSort()

```C
void InsertSort(nodeptr_t *headRef) {
    nodeptr_t result = NULL;
    nodeptr_t current = *headRef;
    nodeptr_t next;
    while (current!=NULL) {
        next = current->next; // 此处有技巧
        SortedInsert(&result, current);
        current = next;
    }   
    *headRef = result;
}
```

### 6. MergeSort()

```C
void MergeSort(nodeptr_t* headRef) {
    nodeptr_t head = *headRef;
    nodeptr_t a;
    nodeptr_t b;

    if ((head == NULL) || (head->next == NULL)) {
    return;
    }
    FrontBackSplit(head, &a, &b);
    MergeSort(&a);
    MergeSort(&b);
    *headRef = SortedMerge(a, b);
}
```

引用的子函数：

```C
/**
 * FrontBackSplit - 将链表对半分为两份
 *
 * @source:     要分解的链表
 * @frontRef:   分解后的第一个链表
 * @backRef:    分解后的第二个链表
 */
void FrontBackSplit(nodeptr_t source, nodeptr_t* frontRef, nodeptr_t* backRef) {
    int len = Length(source);
    int i;
    nodeptr_t current = source;
    if (len < 2) {
        *frontRef = source;
        *backRef = NULL;
    }
    else {
        int hopCount = (len-1)/2; //(figured these with a few drawings)
        for (i = 0; i<hopCount; i++) {
            current = current->next;
        }
        *frontRef = source;
        *backRef = current->next;
        current->next = NULL;
    }
}

/**
 * SortedMerge - 将排序好的链表合并在一起
 *
 * @a: 要合并的链表
 * @b: 也是要合并的链表
 */
nodeptr_t SortedMerge(nodeptr_t a, nodeptr_t b) {
    nodeptr_t result = NULL;

    if (a==NULL) return(b);
    else if (b==NULL) return(a);

    if (a->data <= b->data) {
        result = a;
        result->next = SortedMerge(a->next, b);
    }
    else {
        result = b;
        result->next = SortedMerge(a, b->next);
    }
    return(result);
}

```

### 7. Reverse()

```C
void Reverse(nodeptr_t* headRef) {
    nodeptr_t result = NULL;
    nodeptr_t current = *headRef;
    nodeptr_t next;
    while (current != NULL) {
        next = current->next; // 此处加了特技
        current->next = result;
        result = current;
        current = next;
    }
    *headRef = result;
}

void RecursiveReverse(nodeptr_t* headRef) {
    nodeptr_t first;
    nodeptr_t rest;
    if (*headRef == NULL)
        return;
    first = *headRef;
    rest = first->next;
    if (rest == NULL)
        return;
    RecursiveReverse(&rest);
    first->next->next = first;
    first->next = NULL; // 这里，最好画一下
    *headRef = rest;
}
```