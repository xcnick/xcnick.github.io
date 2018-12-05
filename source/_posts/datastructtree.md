title: 数据结构-树
date: 2018-04-23 10:26:43
tags: DataStruct
mathjax: true
---

> 树的基础概念
<!-- more -->
# 树的基本概念
* 递归定义
* 根节点，儿子，父亲
* 没有儿子的节点称为树叶
* 具有相同父亲节点称为兄弟
* 深度，从根到此节点的唯一路径的长
* 高度，从此节点到一片树叶的最长路径长
* 一颗树的深度等于最深的树叶的深度，该深度等于树的高
* 基本定义
```C
typedef struct TreeNode *PtrToNode
struct TreeNode
{
  ElementType element;
  PtrToNode FirstChild;
  PtrToNode NextSibling;
}
```
* 先序遍历
节点，左子树，右子树
* 中序遍历
左子树，节点，右子树
* 后序遍历
左子树，右子树，节点
* 层序遍历
利用队列的先进先出特点，依次将结点的左、右孩子入队，然后依次出队访问，以此为循环


# 二叉树
* 定义：每个节点不能有多于两个儿子
* 平均深度：$O(\sqrt{N})$
* 定义
```C
typedef struct TreeNode *PtrToNode
typedef struct PtrToNode Tree
struct TreeNode
{
  ElementType element;
  Tree left;
  Tree right;
}
```

## 二叉查找树
* 定义：对于树中的每个节点$X$，其左子树中的所有关键字的值小于$X$关键字的值
* 平均深度：$O(log{N})$

## 平衡查找树(AVL)
* 深度：$O(logN)$
* 每个节点的左子树和右子树高度最多差1