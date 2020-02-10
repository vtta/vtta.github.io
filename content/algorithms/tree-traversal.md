+++
title = "树的遍历"
weight = 1
order = 1
date = 2020-01-22
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]
+++

之前写遍历全都无脑递归，
实在是因为太好写，而且去年哦不前年看邓公讲的东西到现在早给忘光了。

这两天又把邓公视频和PPT看了几遍，实在是妙啊。

## 前序

前序和中序都是把二叉树看成左侧链和右子树。
整体看整个树的遍历是先访问左侧链然后访问某个节点的右子树，
而这个全局规律也适用于要访问的那颗右子树。

是不是有点递归的味道嘿嘿，
那是因为前序和中序虽然不是尾递归，但都有点尾递归的味道。

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200122141034.png)

先顺着左侧链一路访问到底，访问过程中将右子树依次入栈。
左侧链访问完毕后，出栈一个子树然后对这个子树执行一样的操做/循环。
直到整个栈为空。

```cpp
template <typename Fn>
void pre_order(node *x, Fn &f) {
    std::stack<node *> s;
    while (true) {
        // visit along left branch
        while (x != nullptr) {
            f(x->key);
            s.push(x->right);
            x = x->left;
        }
        if (s.empty()) {
            break;
        }
        // switch to the deepest right sub-tree
        x = s.top();
        s.pop();
    }
}
```

## 中序

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200122141013.png)

中序的大致思路差不多。
区别在于沿着左侧链下行的过程中先将节点本身入栈，等到出栈时（即左子树都访问完毕）再访问。
到达左侧最深的节点时，左子树为空，即代表这个不存在的左子树已经访问完毕。
也就是可以访问这个节点本身，完毕后然后再转入右子树访问。

```cpp
template <typename Fn>
void in_order(node *x, Fn &f) {
    std::stack<node *> s;
    while (true) {
        // go along left
        while (x != nullptr) {
            s.push(x);
            x = x->left;
        }
        if (s.empty()) {
            break;
        }
        x = s.top();
        s.pop();
        f(x->key);
        // switch to the right sub-tree
        x = x->right;
    }
}
```

### 中序后继

两种情况：

1. 当前节点有右子树，那么后继节点就是右子树中的最小值。即右子树左侧链的最低点。
2. 当前节点无右子树，那么后继节点就是将当前节点所在子树当作左子树的那个最近节点。

```cpp
// x cannot be nullptr
// the successor of the last node in inorder sequence is nullptr 
node *succ(node *x) {
    if (x->right) {
        x = x->right;
        while (x->left != nullptr) {
            x = x->left;
        }
    } else {
        while (x->parent && x->parent->right == x) {
            x = x->parent;
        }
        x = x->parent;
    }
    return x;
}
```

## 后序

后序邓公的视频没讲，放假在家书也没带回来，于是在GeeksforGeeks上抄了个野鸡版的，没想到还有两种。

### 双栈版

双栈版的比较好想好写，思路是先找到后序的逆序，存放在一个栈`s2`里面，
那么这个栈中元素出栈顺序就是后序遍历的顺序。
那么怎么构造这个序列呢？

这时需要一个辅助栈`s1`。
怎么用这个辅助栈呢？
这个辅助栈起始时只包含根节点。
每次从`s1`中弹出一个节点，展开这个节点。
因为我们要在`s2`中构造后序的逆序，即先自己、再右孩子、再左孩子。
所以这时应该将“自己”，即`s1`弹出的节点压入`s2`。
然后将其左右孩子分别入`s1`，即等待展开其左右孩子。
为什么压入`s2`时是先左后右呢，是因为我们想要先将右子树对应序列加入`s1`。

```cpp
template <typename Fn>
void post_order(node *x, Fn &f)  {
    stack<node *> s1{}, s2{};
    s1.push(x);
    while (!s1.empty()) {
        x = s1.top();
        s1.pop();
        if (x != nullptr) {
            s2.push(x);
            s1.push(x->left);
            s1.push(x->right);
        }
    }
    while (!s2.empty()) {
        f(s2.top()->key);
        s2.pop();
    }
}
```

### 单栈版

思路其实也是左侧链，关键在于如何判断该对节点进行访问。

解释请看[这里](https://www.geeksforgeeks.org/iterative-postorder-traversal-using-stack/)。
其实还有第二种单栈写法，就是入栈时给元素打上标记，不是很喜欢。

要注意的地方是入栈时将右孩子也入栈，
便于判断右子树是否已经被访问过了。

```cpp
template <typename Fn>
void post_order(node *x, Fn &f) {
    std::stack<node *> s;
    while(true) {
        while (x != nullptr) {
            // all pointers in the stack are not nullptr
            if (x->right != nullptr) {
                s.push(x->right);
            }
            s.push(x);
            x = x->left;
        }
        if (s.empty()) {
            break;
        }
        x = s.top();
        s.pop();
        // need to traverse right sub-tree
        if (x->right && !s.empty() && s.top() == x->right) {
            s.pop();
            s.push(x);
            x = x->right;
        } else {
            // both left and right sub-trees are traversed
            f(x->key);
            // root of the sub-tree is traversed
            // need to pop a new node from stack
            x = nullptr;
        }
    }
}
```

## 题目

扯了这么久算法，LeetCode上对应的题目有这么些：
- [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)
- [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
- [144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)
- [145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)
- [230. Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)
- [173. Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)

