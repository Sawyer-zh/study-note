---
title: 669. Trim a Binary Search Tree（leetcode）
date: 2017-09-17 10:43:33
tags:
- leetcode
- 算法
---

### 题目描述

```
Given a binary search tree and the lowest and highest boundaries as L and R, 
trim the tree so that all its elements lies in [L, R] (R >= L). 
You might need to change the root of the tree, 
so the result should return the new root of the trimmed binary search tree.

Example 1:
Input: 
    1
   / \
  0   2

  L = 1
  R = 2

Output: 
    1
      \
       2

Example 2:
Input: 
    3
   / \
  0   4
   \
    2
   /
  1

  L = 1
  R = 3

Output: 
      3
     / 
   2   
  /
 1
```

题目大意就是删除那些不在范围内的节点，同时保证这棵树还是一棵二分搜索树

### 代码实现

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int L, int R) {
    	if (NULL == root) {
    		return NULL;
    	}

    	if ((root->val >= L) && (root->val <= R)) {
    		root->left = trimBST(root->left, L, R);
    		root->right = trimBST(root->right, L, R);
    		return root;
    	}

    	if (root->val < L) {
    		return trimBST(root->right, L, R); // 如果这个节点小于L，那么这个节点和它的左子树都被丢弃
    	}
    	if (root->val > R) {
    		return trimBST(root->left, L, R); // 如果这个节点大于R，那么这个节点和它的右子树都被丢弃
    	}
    }
};
```

**关键就是利用二叉树的性质：左子树中所有的节点都比父节点小，右子树所有的节点都比父节点大。**