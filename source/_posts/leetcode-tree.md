title: LeetCode树总结
date: 2018-11-24 09:30:24
tags: LeetCode
---

> 树题目总结
<!-- more -->
# 二叉树的定义
```Cpp
struct TreeNode {
  int val;
  TreeNode *left;
  TreeNode *right;
  TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

二叉树的四种遍历方式：前序遍历、中序遍历、后续遍历和层次遍历，其中前三种为一类，一般是使用栈（stack）实现，三种遍历方式的区别在于访问根结点的顺序不同；最后一种一般使用队列（queue）实现。

具体来说

1. 前序遍历：访问顺序是根结点->左孩子->右孩子。迭代算法实现的主要思想为：先访问根结点，然后若右孩子存在就将其存入栈中，若左孩子存在，则根结点沿左孩子一直到叶结点同时依次访问；到了叶结点时， 去除栈顶结点继续访问，直到栈为空。

```Cpp
// 迭代
vector<int> preorder(TreeNode *root) {
  vector<int> res;
  if (!root) {
    rturn res;
  }
  stack<TreeNode *> s;
  s.push(root);
  while (!s.empty()) {
    res.push_back(root->val);
    if (root->right) {
      s.push(root->right);
    }
    if (root->left) {
      root = root->left;
    } else {
      root = s.top();
      s.pop();
    }
  }
  return res;
}

// 递归
vector<int> preorder(TreeNode *root) {
  vector<int> res;
  preorderTra(root, res);
  return res;
}

void preorderTra(TreeNode *root, vector<int> &res) {
  if (!root) {
    return;
  }
  res.push_back(root->val);
  preorderTra(root->left, res);
  preorderTra(root->right, res);
}
```

2. 中序遍历：访问顺序是左孩子->根结点->右结点。迭代算法实现的主要思想为：因为先访问左孩子（从身为叶结点的左孩子开始），所以可以利用栈先进后出的特点，我们先沿着左孩子一直将结点存入栈中，到叶结点时，从栈中取出结点开始访问，然后取当前结点的右孩子接着进行循环，直到root为NULL且栈为空。

```Cpp
// 迭代
vector<int> inorder(TreeNode *root) {
  vector<int> res;
  stack<TreeNode *> s;
  while (root || !s.empty()) {
    if (root) {
      s.push(root);
      root = root->left;
    } else {
      root = s.top();
      res.push_back(root->val);
      s.pop();
      root = root->right;
    }
  }
  return res;
}

// 递归
vector<int> inorder(TreeNode *root) {
  vector<int> res;
  inorderTra(root, res);
  return res;
}

void inorderTra(TreeNode *root, vector<int> &res) {
  if (!root) {
    return;
  }
  inorderTra(root->left, res);
  res.push_back(root->val);
  inorderTra(root->right, res);
}
```

3. 后序遍历：访问顺序是左孩子->右孩子->根结点。迭代算法实现的主要思想为：最重要的是定义一个指针记录下前一个访问过的，这样可以避免重复访问。有两种思路：一、按照后序遍历访问结点的顺序，将右、左孩子入栈（注意入栈顺序），若到达根结点或当前结点的左、右孩子中已被访问过，则从栈中取出结点访问。二、可以从中序遍历的基础上改进，即不断将左孩子压入栈中，然后若当前结点的右孩子没有被访问过，则转向遍历以当前右孩子为根结点的树，若访问过，则取出栈顶元素访问即可。这两种思路中都要注意对前指针的更新。

```Cpp
// 迭代
vector<int> postorder(TreeNode *root) {
  vector<int> res;
  stack<TreeNode *> s;
  if (root) {
    s.push(root);
  }
  TreeNode *head = root;
  while (!s.empty()) {
    TreeNode *top = s.top();
    // 两种情况：1. 到达叶节点；2.前一元素是栈顶的左右孩子，被访问过
    if ((!top->left && !top->right) || top->left == head ||
        top->right == head) {
      res.push_back(top->val);
      s.pop();
      head = top();
    } else {
      if (top->right) {
        s.push(top->right);
      }
      if (top->left) {
        s.push(top->left);
      }
    }
  }
  return res;
}

// 递归
vector<int> postorder(TreeNode *root) {
  vector<int> res;
  postorderTra(root, res);
  return res;
}

void postorderTra(TreeNode *root, vector<int> &res) {
  if (!root) {
    return;
  }
  postorderTra(root->left, res);
  postorderTra(root->right, res);
  res.push_back(root->val);
}
```

4. 层序遍历：利用队列先进先出的特点，依次将结点的左、右孩子入队，然后依次出队访问，以此为循环。
迭代的方法分为3种：
* 定义两个计数器：一个记录本层结点数，一个记录下当前层下一层的结点数，这样遍历完一层后输出，然后转向下一层，并更新结点数。
* 只需一个计数器，记录下当前行结点数，使用两层while循环结构，每访问完一个结点，计数器减1，直到该层中所有结点访问；然后访问下一行，并通过size()得到该行的结点数，以此进行循环.
* 在每一层的最后向栈中压入NULL当作该行的结束符，这样出队时，每遇到NULL时，就保存这一行中的结点，值得注意的时，只有队列不为空时才压入NULL。

```Cpp
vector<vector<int>> levelOrder(TreeNode* root) {
  vector<vector<int>> res;
  if (!root) {
    return res;
  }
  queue<TreeNode*> q;
  q.push(root);
  while (!q.empty()) {
    size_t size = q.size();
    vector<int> onelevel;
    while (size-- != 0) {
      TreeNode *node = q.front();
      q.pop();
      onelevel.push_back(node->val);
      if (node->left) {
        q.push(node->left);
      }
      if (node->right) {
        q.push(node->right);
      }
    }
    res.push_back(onelevel);
  }
  return res;
}
```

# 二叉树的最大深度

> 给定一个二叉树，找出其最大深度。
二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

* 使用深度优先搜索DFS，递归方法。

```Cpp
int maxDepth(TreeNode *root) {
  if (!root) {
    return 0;
  }
  return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```

* 使用层序遍历二叉树，计算总层数，即为二叉树最大深度。

```Cpp
int maxDepth(TreeNode *root) {
  if (!root) {
    return 0;
  }
  int res = 0;
  queue<TreeNode *> q;
  q.push(root);
  while (!q.empty()) {
    ++res;
    int n = q.size();
    for (int i = 0; i < n; ++i) {
      TreeNode *t = q.front();
      q.pop();
      if (t->left) {
        q.push(t->left);
      }
      if (t->right) {
        q.push(t->right);
      }
    }
  }
  return res;
}
```

# 验证二叉搜索树

> 给定一个二叉树，判断其是否是一个有效的二叉搜索树。
假设一个二叉搜索树具有如下特征：
节点的左子树只包含小于当前节点的数。
节点的右子树只包含大于当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

二叉搜索树：若是所有结点互不相等，满足：在任一结点r的左（右）子树中，所有结点（若存在）均小于（大于）r。更一般性的特点是：任何一棵二叉树是二叉搜索树，当且仅当其中序遍历序列单调非降。

* 中序遍历。比较相邻两个结点值的大小，看是否为非降型。

```Cpp
bool isValidBST(TreeNode *node) {
  stack<TreeNode *> s;
  TreeNode *pre = root;
  TreeNode *cur = root;

  while (cur || !s.empty()) {
    if (cur) {
      s.push(cur);
      cur = cur->left;
    } else {
      cur = stk.top();
      s.pop();
      if (pre && cur->val < pre->val) {
        return false;
      }
      pre = cur;
      cur = cur->right;
    }
  }
  return true;
}
```

* 整体递归

```Cpp
bool isValidBST(TreeNode *root) {
  return isValid(root, INT64_MIN, INT64_MAX);
}

bool isValid(TreeNode *root, long mn, long mx) {
  if (!root) {
    return true;
  }

  if (root->val <= mn || root->val >= max) {
    return false;
  }
  return isValid(root->left, mn, root->val) && isValid(root->right, root->val, mx);
}
```