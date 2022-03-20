---
layout:   post
title:   "二分查找代码模板"
subtitle:  "binary search"
date:    2021-07-08 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

二分查找是一种用于在有序数组里查找对应元素的算法。



### 二分查找与dfs和bfs搜索的区别

搜索可以认为是对整个空间对一种搜索，但是查找类的算法，比如说二分查找，它不一定会对整个空间做搜索，根据数据的有序性，结合具体的算法，能优化数据查询的时间和空间复杂度。



### 二分查找算法分析

#### 满足一下三个条件：

- 函数具备单调性(单调递增或单调递减)
- 函数是有界
- 具备随机访问的特性

前面2条可以总结为单调有界，第三条是方便计算机的程序处理。



#### 复杂度

二分查找相比暴力查找而言，速度的确是快了等级，时间复杂度是o(logn)，其中n是数组的长度，空间复杂度一般是o(1)。



#### 具体的算法过程

这个算法不难，下面的这个图可以形象的描述一下具体的过程：从两边向中间收敛的这么一种状态。
![](/img/in-post/zhouhaijun-pic/binary_search_1.png)

假设要查找的数据不在数组里，那么遍历的最后一个状态是Low=High=Mid，这是一个细节，在写代码的时候需要注意下。



### 代码模板

```c++
int binarySearch(const vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
        		// 出口1:找到了元素，返回数组下标
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    // 出口2: 没找到元素，返回-1
    return -1;
}
```

有两处细节需要注意一下：

- 有两个出口

两个出口都在上述代码模板做了标记

- 向上或向下取整

向上取整，一般是通过上述代码的出口2最终返回左边界；如果是向下取整，是通过出口2返回右边界。



### 总结

在单调有界的数组里查找元素，用二分查找算法是一种很明智的选择，相比于暴力查找而言在时间复杂度上降低了一个等级，常见的有：求一个数的平方根、在一个有序或部分有序数组里查找元素等。

### leetcode题目

[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

[x的平方根](https://leetcode-cn.com/problems/sqrtx/)

[搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

[搜索旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

