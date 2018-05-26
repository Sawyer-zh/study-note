---
title: 496. Next Greater Element（leetcode）
date: 2017-09-19 14:34:08
tags:
- leetcode
- 算法
---

### 题目描述

You are given two arrays (without duplicates) nums1 and nums2 where nums1’s elements are subset of nums2. 
Find all the next greater numbers for nums1's elements in the corresponding places of nums2.

The Next Greater Number of a number x in nums1 is the first greater number to its right in nums2. 
If it does not exist, output -1 for this number.

简单的说就是对于第一个数组中的每一个数字，我们都去第二个数组里面找，直到找到第一个比它大的数字，然后放入一个容器里面。如果没有找到，就把-1放入容器里面。

    Input: nums1 = [4,1,0], nums2 = [1,0,4,2].
    Output: [-1,4,4]
        Explanation:
        For number 4 in the first array, 
        you cannot find the next greater number for it in the second array, 
        so output -1.
    
        For number 1 in the first array, 
        the next greater number for it in the second array is 4.
    
        For number 0 in the first array, 
        there is no next greater number for it in the second array, 
        so output 4.

    Input: nums1 = [2,4], nums2 = [1,2,3,4].
    Output: [3,-1]
    Explanation:
        For number 2 in the first array, 
        the next greater number for it in the second array is 3.
    
        For number 4 in the first array, 
        there is no next greater number for it in the second array, 
        so output -1
### 暴力破解

暴力破解，也就是没有利用数据之间的规律。毫无技巧可言，只需要细心一些即可。

```c++
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& findNums, vector<int>& nums) {
        std::vector<int>::iterator iter;
        std::vector<int> v;
        for (std::vector<int>::iterator i = findNums.begin(); i != findNums.end(); ++i) {
        	iter = find(nums.begin(), nums.end(), *i);
            int front = *iter;
            int next = *(++iter);
            --iter;
        	if ((iter != nums.end()) && (iter != --nums.end())) {
        		std::vector<int>::iterator j;
        		for (j = iter; j != nums.end(); ++j) {
        			if (*j > *iter) {
                        v.push_back(*j);
                        break;
        			}
        		}
        		if (j == nums.end()) {
        			v.push_back(-1);
        		}
        	}
        	else {
        		v.push_back(-1);
        	}
        }

        return v;
    }
};
```

(建议对vector容器的遍历用下标来访问，而不是我上面的那样用迭代器来操作)

可以看出，这样的代码虽然可以解出来，但是太烂了，看起来和那个啥一样。所以我们就需要一种新的思路。

### 优化方案一

首先，我们仔细分析一下题目。题目说了**第一个数组是第二个数组的子集**对吧。那么，这就意味着我**一定可以在第二个数组里面找到第一个数组中的元素**。正是因为有了这个一定可以找到的特点，我们可以使用哈希表。让第二个数组中的元素值和它所在的位置做一个映射。这样做了之后，当我们需要在第二个数组里面查找第一个数组中的数据的时候，我们就可以以**O(1)的时间复杂度**找到这个元素在第二个数组中的位置。然后，我们只需要对它后面的元素进行遍历即可。

```c++
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& findNums, vector<int>& nums) {
        vector<int> res(findNums.size());
        unordered_map<int, int> m;
        for (int i = 0; i < nums.size(); ++i) {
            m[nums[i]] = i; // 做一个映射的关系
        }
        for (int i = 0; i < findNums.size(); ++i) {
            res[i] = -1;
            int start = m[findNums[i]];
            for (int j = start + 1; j < nums.size(); ++j) {
                if (nums[j] > findNums[i]) {
                    res[i] = nums[j];
                    break;
                }
            }
        }
      
        return res;
    }
};
```

这样的话，是不是代码更清晰易懂了？

### 优化方案二

在优化方案一中，我们是利用哈希表在第二个数组中找第一个数组中的元素。但是我们并没有立马得出这个元素的后面出现的第一个比这个元素大的元素，那么我们可以想，是否能够利用哈希表来直接得到这个元素后面出现的第一个比这个元素大的元素呢？

我们的做法是把第二个数组中的元素和该元素后面出现的第一个最大值做一个映射。

那么为了减少无效的遍历（因为，这个元素前面的元素是没有意义的。例如：有个数组`[1,3,4,2]`，当我们去查找4这个元素后面出现的第一个比4大的元素的时候，我们去遍历1和3是没有意义的。我们称之为无效的遍历）。所以我们使用栈这个数据结构帮我们解决掉无效的遍历，简化了搜索空间。

```c++
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& findNums, vector<int>& nums) {
        vector<int> res;
        stack<int> st;
        unordered_map<int, int> m;
        for (int num : nums) {
            while (!st.empty() && st.top() < num) {
                m[st.top()] = num; st.pop();
            }
            st.push(num);
        }
        for (int num : findNums) {
            res.push_back(m.count(num) ? m[num] : -1);
        }        
        return res;
    }
};
```

### 总结

所以，遇到这种查找一个元素是否在这个数组（数组中的元素不重复）里面的问题，我们可以用哈希表来处理。首先对这个数组进行元素和位置的映射，然后，无论你要查找哪个元素，我都可以轻而易举的找到，而不用每次都去遍历来查找。

栈这种数据结构可以寻找相邻的更大或者更小的数字，简化搜索空间。





