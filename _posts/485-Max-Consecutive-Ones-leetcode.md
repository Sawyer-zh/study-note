---
title: 485. Max Consecutive Ones (leetcode)
date: 2017-09-23 10:42:07
tags:
- 算法
- leetcode
---

### 题目描述

Given a binary array, 
find the maximum number of consecutive 1s in this array.

Example 1:
Input: [1,1,0,1,1,1]
Output: 3
Explanation: The first two digits or the last three digits are consecutive 1s.

The maximum number of consecutive 1s is 3.

Note:

The input array will only contain 0 and 1.
The length of input array is a positive integer and will not exceed 10,000

题目大概的意思就是**找到最多有几个连续的1**

这题挺水的，但是开始的时候，我想了一下用**栈**这个数据结构，毕竟栈有个很重要的功能就是可以**反悔**，并且删除元素也很快，`pop`一下就好了。而在这题，需要反悔的地方就是如果有一个更多连续的1，那么，把连续1的个数压栈。

```c++
class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
    	int n = nums.size();
    	stack<int> s;
    	int sum_max = 0;

        for (int i = 0; i < n;i++) {
            if (nums[i] == 1) {
            	sum_max++;
            }
            else if (nums[i] == 0) {
                if (!s.empty()) {
                	if (sum_max > s.top()) {
                		s.pop();
                		s.push(sum_max);
                	}
                }
                else {
                	s.push(sum_max);
                }

                sum_max = 0;
            }
        }

        if (!s.empty()) {
        	if (sum_max > s.top()) {
        		return sum_max;
        	}
        	else {
                return s.top();
        	}
        }
        else {
        	return sum_max;
        }
    }
};
```

可以看到，这段代码很难看？因为是有太多的`if 和 else`了，原因就是我需要**判断栈是否为空导致**。而且，我在交换比较最大值的时候，也用了`if else`，实际上直接`Max = max(Max, otherNum); `即可。

所以需要换了思路

```c++
class Solution  
{  
public:  
    int findMaxConsecutiveOnes(vector<int>& nums)  
    {  
        int Max = 0, cnt = 0;  
        for(auto n:nums)  
        {  
            if(n == 1)  
            {  
                ++cnt;  
            }  
            else  
            {  
                Max = max(Max, cnt);  
                cnt = 0;  
            }  
        }  
        Max = max(Max, cnt);  
        return Max;  
    }  
};
```

### 总结

1、栈、队列等等数据结构在存放数据量很小的时候，可以不用使用。

2、比较两个最大值的时候，使用`max()`函数，不要自己写判断语句，显得代码难看。