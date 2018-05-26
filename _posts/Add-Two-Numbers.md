---
title: ' Add Two Numbers'
date: 2017-09-16 15:01:54
tags:
- 算法
- leetcode
---

### 题目描述

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Input:** (2 -> 4 -> 3) + (5 -> 6 -> 4)
**Output:** 7 -> 0 -> 8

**Input:** (1 -> 8) + (0)
**Output:** 1 -> 8

题目的意思就是模拟两个数的加法运算

### 思路

1、使用一个新的链表来存放这些加好的数字。

2、先让两个数字对应的部分相加，然后再加上上一次相加的进位（如果没有进位，那么进位实际上就是0），得到一个新的数字。

3、如果这个数字大于9，那么就要对这个数字进行进位了。其实就是分出个位和十位。保留个位，而十位留给下一次相加的数。

4、关键是要考虑两个链表不一样长的情况（也就是数字的位数不同的情况）。为了好操作，我们可以**把短的链表用0来填充**。而且由于链表的长度不一样，很容易会让本来就是NULL的节点往后面指。

### 代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    	ListNode *headNode = new ListNode(0), *currentNode = headNode;
    	ListNode *p = l1, *q = l2; // 一般我们不会去直接操作传过来的参数，所以另外保存它们
    	int carry = 0, x, y, sum;
    	while (p != NULL || q != NULL) {
    		x = (p != NULL) ? p->val : 0;
    		y = (q != NULL) ? q->val : 0;
            
            sum = x + y + carry;
            currentNode->next = new ListNode(sum % 10);
            currentNode = currentNode->next;
            carry = sum / 10;

            if ((p != NULL) && (p->next != NULL)) {
            	p = p->next;
            }
            else {
            	p = NULL;
            }

            if ((q != NULL) && (q->next != NULL)) {
            	q = q->next;
            }
            else {
            	q = NULL;
            }
    	}

    	if  (carry > 0) {
    		currentNode->next = new ListNode(carry);
    	}

    	return headNode->next;
    }
};
```







