---
title: 详解二叉树（基础与BST）
date: 2018-03-22 21:22:53
tags:
    - C
categories: 数据结构
---

二叉树是一种树。这个树的每个节点都有不超过两个节点的孩子。我们通常将二叉树倒画，即将其根画在顶部，叶子画在最底部。一颗典型的二叉树表示如下：

![](https://coding.net/u/name1e5s/p/pic/git/raw/master/binarytree.gif)

现在给出二叉树的正式递归定义：

> A binary tree is either empty (represented by a null pointer), or is made of a single node, where the left and right pointers (recursive definition ahead) each point to a binary tree. 

> 二叉树要么是一颗空树（使用一个 NULL 指针表示），要么左右孩子都指向一颗二叉树。

<!--more-->

“二叉搜索树（BST）” 或者说是有序二叉树，是二叉树的一种。在 BST 中，一个节点的所有左孩子存储的数据都比节点自身存储的数据小（或者相等），与之对应的，右孩子存储的数据都比节点自身要大。上图其实就是二叉搜索树的示例。在上图中，根节点存储的数据是 5，左孩子存储的 {1,3,4} 都比 5 小，右孩子存储的数据 {6,9} 都比 5 大。该关系可以递归下去，直到树叶。

要注意二叉树和二叉搜索树的区别，一词之差，所指的东西就完全不一样了。

## 二叉搜索树的使用场景

二叉搜索树的查找和插入操作都很快。将在后面看到这两个算法的代码。二叉搜索树查找一个节点的位置的平均时间复杂度为 O(log N)。因此该数据结构适用于需要多次插入/查找数据的场景。不过要注意，查找算法不总是 O(log N) 的，有时会慢上很多。

## 树的类型

常用的树的类型有以下几种：

1. 满二叉树：任意一个节点只有零或两个子节点的树叫做满二叉树，示例如下：

```
               18
           /       \  
         15         30  
        /  \        /  \
      40    50    100   40

             18
           /    \   
         15     20    
        /  \       
      40    50   
    /   \
   30   50

           18
        /     \  
      40       30  
               /  \
             100   40
```

2. 完全二叉树：若设二叉树的深度为h，除第 h 层外，其它各层 （1~h-1） 的结点数都达到最大个数，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。

```
               18
           /       \  
         15         30  
        /  \        /  \
      40    50    100   40


               18
           /       \  
         15         30  
        /  \        /  \
      40    50    100   40
     /  \   /
    8   7  9 
```
3. 完美二叉树：满足完全二叉树性质，树的叶子节点均在最后一层的二叉树。示例如下：

```
               18
           /       \  
         15         30  
        /  \        /  \
      40    50    100   40


               18
           /       \  
         15         30  
```

4. 平衡二叉树：具有 N 个节点且高度为 O(log N) 的二叉树。例如 AVL 树，红黑树。会在后部分详细解释。

5. 退化的树，只有一边孩子的树。和链表类似。

[Ref: [Wikipedia](https://en.wikipedia.org/wiki/Binary_tree#Types_of_binary_trees)]
## 理解本文的方法

本系列文章提到的问题有时仅仅需要一个基本的二叉树，有的则需要二叉搜索树。无论在哪种情况下，我们关注的都是指针和递归的结合（当然二叉树也可以使用数组实现，但是这不在本文的讨论范围之内）。也就是说，在理解本文之前，你需要确保你对 C 语言的指针有着足够的了解。对于文中提到的每一个问题，你需要注意的有：

1. 构成二叉树的节点和指针，以及操作二叉树的代码

2. 遍历二叉树的算法（通常是递归算法）

当你考虑有关二叉树的问题时，先画出一些小的树来理解问题是一个很好的选择。

## 树的结构

在 C 语言中，树的实现和链表的实现有些类似。都是数据区加上指针区。一个典型的树的声明如下：

```C
struct node {
    int             data;
    struct node     *left;
    struct node     *right;
}

typedef struct node node_t;
typedef struct node* nodeptr_t;
```

本文多数时候使用的声明也是如此。在真正应用时，结构体内有时还会包括关于父节点的信息，或者是用于平衡树的信息。

一般情况下，如果某一个节点的子节点不存在，我们就使用 NULL 来标记。

## 树的遍历

树的遍历操作有三种，前序遍历，中序遍历和后序遍历。三者的不同之处在于处理子节点的时间不同。前序遍历是先处理根节点，然后处理左孩子最后处理右孩子。将处理根节点的操作挪到处理左右节点的操作之间，我们就得到了中序遍历。如果挪到最后，那就是后序遍历。示例如下：

```C
void PrintTree_PreOrder(nodeptr_t root) {
    if(root) {
        printf("%d\n",root->data);
        PrintTree_PreOrder(root->left);
        PrintTree_PreOrder(root->right);
    }
}
```

--------------------

练习：
    写出上述程序的中序遍历以及后续遍历版本。

--------------------

可以看到，我们在遍历树时多次应用了递归。这是由于树的递归定义决定的特点。下面再给出一个遍历的例子：

```C
int treeSize(nodeptr_t root) {
    if(root == 0) {
        return 0;
    }
    return 1 + treeSize(root->left) + treesize(root->right)
}
```

## 二叉搜索树

在文章的开头，我们介绍了二叉搜索树的定义。现在，我们来详细看下其实现。

### 二叉树的查找

如前文所述，二叉树的左右孩子的大小是有限制的。因此我们可以很简便的写出搜索数据的函数。

```C
nodeptr_t treeSearch(nodeptr_t root, int value) {
    if(root == NULL)
        return NULL;
    else if(root->data == value)
        return root;
    else if(root->data > target)
        return treeSearch(root->left);
    else
        return treeSearch(root->right);
}
```

为了防止爆栈这种悲剧发生（概率很低），我们还可以写出迭代版本的搜索算法。示例如下：

```C
nodeptr_t treeSearch(nodeptr_t root, int value)
{
    while(root != NULL && root->data != value) {
        if(root->data > value) {
            root = root->left;
        } else {
            root = root->right;
        }
    }

    return root;
}
```

为了处理其他类型的数据，我们还可以把比较操作那里使用函数进行替代。比如我们在实现字典的查找时，就可以简单地使用 `strcmp` 函数进行比较操作。

### 插入新节点

插入新节点的最简单方法就是一路搜索下去，直到搜索到一个还没有节点的位置，之后进行插入。实现如下：

```C
void treeInsert(nodeptr_t root, int data)
{
    nodeptr_t newNode;

    newNode = malloc(sizeof(*newNode));
    assert(newNode);

    newNode->data = data;
    newNode->left = 0;
    newNode->right = 0;

    for(;;) {
        if(root->data > data) {
            if(root->left) {
                root = root->left;
            } else {
                root->left = newNode;
                return;
            }
        } else {
            if(root->right) {
                root = root->right;
            } else {
                root->right = newNode;
                return;
            }
        }
    }
}
```

这种操作的实现是极其简洁的。但是其缺点也是很明显的：这钟插入操作没有尝试平衡树。也就是说，在最坏的情况下，树可能只向某一个方向生长，使之退化为链表。我们会在后面引入改进的树结构来解决问题。

### 删除节点

基于树的特性，我们有时可以通过一种不使用 free() 的操作来进行杉树操作，步骤如下：

1. 在树的结构体内加入一个变量指示词节点是否被删除。

2. 要删除某一节点，改变指示变量即可。

操作极其简单，在此不再赘述。

现在我们来看“真正的删除树的操作”。要删除一个节点，我们需要考虑以下几种情况：

1. 要删除的节点没有子节点，此时的操作极为简单，只需要 `free` 掉该节点即可。

2. 要删除的节点有一个子节点，此时的操作也不算复杂，只需要将其父节点指向其的指针替换为其子节点的指针，之后 `free` 该节点。

3. 要删除的节点有两个节点，这种情况就比较复杂了。一般的做法是使用其右子树的最小节点的数据替换掉该节点的数据，然后递归的删除掉右子树的最小节点。

实现如下：

```C
void treeDelete(nodeptr_t root, int value) {
    nodeptr_t temp;

    if(root == NULL)
        return;
    else if(value < root->data)
        treeDelete(root->left,data);
    else if(value > root->data)
        treeDelete(root->right,data);
    else if(root->left && root->right) {
        temp = __findMin(root->right);
        root->data = temp->data;
        treeDelete(root->right,root->data);
    } else {
        temp = root;
        if(root->left == NULL)
            root = root->right;
        else if(root->right == NULL)
            root = root->left;
        free(temp);
    }
}

nodeptr_t __findMin(nodeptr_t root) {
    if(root == NULL)
        return NULL;
    else if(root->left == NULL)
        return root;
    else
        return __findMin(root->left);
}
```

为了实现的易于理解，本代码的性能可能不是很好。对于初学者来说，优化本程序是一个不错的练习。

练习：

写出销毁整棵树的函数，声明如下：

```C
/**
 * destroyTree - ...
 *
 * @rootptr:    pointer to the root of the tree.
 */
void destroyTree(nodeptr_t *rootptr);
```