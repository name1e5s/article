---
title: 详解二叉堆（基础篇）
date: 2018-03-29 21:22:53
tags:
    - C
categories: 数据结构
---

堆(heap)，又称为优先队列(priority queue)。尽管名为优先队列，但堆并不是队列。在队列中，我们可以进行的操作是向队列中添加元素和按照元素进入队列的选后顺序取出元素。而在堆中，我们不是按照元素进入队列的先后顺序，而是按照元素的优先级取出元素。

Linux 内核中的调度器会根据各个进程的优先级对程序的执行进行调度。在操作系统运行时，通常会有很多个不同的进程，优先级各不相同。在调度器的作用下，优先级高的进程被有限执行，优先级靠后的就只能等待。堆是实现这种调度器的一种很合适的数据结构（顺便提一下，现在的 Linux 内核的调度器使用的是基于红黑树的 [CFS](https://en.wikipedia.org/wiki/Completely_Fair_Scheduler) ，笔者以后会专门介绍）。

<!--more-->

## 二叉堆的概念

我们常用的二叉堆就是一颗任意节点的优先级不小于其子节点的完全二叉树。

完全二叉树的定义如下：

> 若设二叉树的高度为h，除第 h 层外，其它各层 (1～h-1) 的结点数都达到最大个数，第 h 层从右向左连续缺若干结点，这就是完全二叉树。

比如下图就是一颗完全二叉树：

```
           10
         /     \            
      15        30  
     /  \      /  \
   40    50  100   40
```

现在假设保存的数值越小的节点的优先级越高，那么上图就是一个堆。我们将任意节点不大于其子节点的堆叫做最小堆或小根堆，将任意节点不小于其子节点的堆叫做最大堆或大根堆。因此，上图就是一个小根堆。

## 二叉堆的实现

身为优先队列，肯定要支持如下两个操作：

1. 插入数据

2. 取出优先级最高的数据

因为完全二叉树的结构很是整齐，且极少有人类能玩转指针，我们的堆通过数组来实现。当使用数组实现时，堆的节点之间有如下关系（假设根节点的索引为0）：

1. 索引为i的左孩子的索引是 2i

2. 索引为i的左孩子的索引是 2i+1

3. 索引为i的父结点的索引是 i/2

根节点为 0 时的节点关系很容易依此推出。

本文中，笔者使用根节点索引为 1 的方式来实现最小堆。数组索引为 0 的地方记录了堆中元素的数目。

### 插入

插入时，我们首先将要插入的数据放在数组的尾部。但是这样破坏了堆的特性，因此我们需要进行调整，保证堆的特性。调整操作如下：

1. 将刚插入的节点和其父节点比较，如果符合堆的形成条件或者已经是根节点，那么堆的插入操作就算结束。

2. 重复执行上一步。

这个操作通常被称为 Percolation Up，图示如下：

![](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Binary%20Heaps/pix/insert.bmp)

```C
void insert_data(int *heap,int value) 
{
    heap[0] = heap[0] + 1;
    heap[heap[0]] = val;
    
    percolate_up(heap);
}

void percolate_up(int *heap) {
    int lightIdx, parentIdx;
    lightIdx  = heap[0];
    parentIdx = lightIdx >> 1;

    while((parentIdx > 0) && (heap[lightIdx] < heap[parentIdx])) {
        swap(heap + lightIdx, heap + parentIdx); 
        lightIdx  = parentIdx;
        parentIdx = lightIdx >> 1;
    }
}
```

`swap()` 函数就是很常见的交换两个值的函数，实现如下：

```C
inline void swap(int *a,int *b) {
    int temp;
    temp = *a;
    *a = *b;
    *b = temp;
    return;
}
```

### 取出优先级最高的元素

取出优先级最高的数据也是同理。我们要做的操作如下：

1. 读取根节点的数据

2. 使用最后一个叶节点的数据替换根节点的数据

3. 将最后的叶节点（即现在的根节点）不断的和子节点比较。如果其比两个子节点中小的那一个大，则和该子节点交换。直到该节点不大于任一子节点都小或成为叶节点。

与上文的 Percolation Up 相对，本节的步骤 3 被称为 Percolating Down。

实现如下：

```C
int delete_min(int *heap) 
{
    int min;
    if (heap[0] < 1) {
        printf("Delete Min: Empty Heap!!!\n");
        return -1;
    }

    min = heap[1];
    swap(heap + 1, heap + heap[0]);
    heap[0] --;

    percolating_down(heap);
 
    return min;
}

void percolating_down(int *heap) {
    int heavyIdx;
    int leftChildIdx, rightChildIdx, minIdx;
    int flag = 1; // Swap ? 1: yes; 0: nope

    heavyIdx = 1;
    while(flag == 1) {
        flag     = 0;
        leftChildIdx = heavyIdx << 1;
        rightChildIdx = leftChildIdx + 1;
        if (leftChildIdx > heap[0]) {
            // both children are null
            break; 
        }
        else if (rightChildIdx > heap[0]) {
            // right children is null
            minIdx = leftChildIdx;
        }
        else {
            minIdx = heap[leftChildIdx] < heap[rightChildIdx] ？ lefiChildIndex : rightChildIndex;
        }

        if (heap[heavyIdx] > heap[minIdx]) {
            swap(heap + heavyIdx, heap + minIdx);
            heavyIdx = minIdx;
            flag = 1;
        }
    }
}
```
## 二叉堆的应用

###堆排序：

所谓堆排序就是使用堆这一数据结构进行的排序操作。我们只需要建一个堆，之后不断输出优先级最高的数据即可完成排序。时间复杂度 O(log N)。

## 二叉堆的相关资料 

[这里](https://visualgo.net/zh/heap) 可以看到堆的可视化。

[陈越姥姥的题目](https://www.patest.cn/contests/pat-a-practise/1098) 是个不错的基础练习。

[POJ 3784](http://poj.org/problem?id=3784) 是一道很不错的基础练习题。

（提示：新建一个大根堆和一个小根堆，保证大根堆里面的数小于小根堆里的数，这样大根堆的堆顶即为中位数。出现新的数字时，只需与之比较即可）。