---
layout:   post
title:   "算法排序(快排)"
subtitle:  ""
date:    2022-11-13 09:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

### 算法简介

- 算法核心思想

以升序排列为例，找一个基准值，让所有左边的元素小于等于该基准值，让右边的元素大于或等于该基准值；
如此一来，整个区间被分成两段：左半段和右半段，即左区间小于等于右区间，最后采用递归的方式分别对左区间、右区间执行同样的排序动作。

相较于选择排序法，快排采用分区的方式优化了元素间的比较次数，某种程度上而言，快排也是一种基于选择排序法的一种剪枝优化。

它的平均时间复杂度是O(n*logN)，最坏的情况下是O(N^2)，导致快排算法不稳定的一个重要因素在于分区，比如说对一个已经排好序的升序数组做快排，这种情况下因为区间只有一边，分区没有效果，算法复杂度退化为O(N^2)，而且这种情况下因为有递归调用，反而增加了一些时间开销。

所以在快排的设计思路里边，分区是一个很重要的概念，在代码设计时应要考虑降低算法退化的可能性，尽可能的让算法效率趋于稳定。



- 动图展示

![](/img/in-post/zhouhaijun-pic/quick-sort-demo.gif)



#### 代码演示基本功能

- 试图来写一个代码，完成快排的基本功能：

```c++
#include <vector>
#include <iostream>

// define the interval rule: left is close and right is open ,e.g. [from, end)
int Partition(std::vector<int>& data, int from, int end) {
  int pivot = end - 1;
  int l = from, r = end - 1;

  while (l < r) {
    // try to find the first element which bigger than pivot from left to right
    while (l < r && data[l] <= data[pivot]) ++l;

    // try to find the first element which littler than pivot from right to left
    while (l < r && data[r] >= data[pivot]) --r;
    
    // check l and r, if l < r, then swap
    if (l < r) {
      std::swap(data[l], data[r]);
    }

  }

  // whatever ,the last status will be: l == r
  // so swap l and pivot, then return l
  std::swap(data[l], data[pivot]);
  return l;
}

void QuickSort(std::vector<int>& data, int from, int end) {
  // terminator for loop ending
  if (from >= end) return;

  int pi = Partition(data, from, end);
  QuickSort(data, from, pi);
  QuickSort(data, pi + 1, end);
}

int main(void)
{
  std::vector<int> data {2,3,5,1,65,78,30,29};
  std::cout << "before sort:" << std::endl;
  for (auto d : data) {
    std::cout << d << " " ;
  }
  std::cout << std::endl;

  QuickSort(data, 0, data.size());


  std::cout << "after sort:" << std::endl;
  for (auto d : data) {
    std::cout << d << " " ;
  }
  std::cout << std::endl;

  return 0;
}
```



- 代码优化

  上述代码存在几个问题：

  - 未实现泛型，主要体现在两个方面：一是采用了vector容器，二是限制了元素的类型，这个问题比较好解，用c++的模板就可以解决；
  - 开篇提到的算法不稳定的问题，因为始终用第一个元素做基准值，极端的情况下算法会退化，要解决这个问题，可以用三端中值法，即取首、中、尾的中间元素作为基准值。



尝试提供一个优化后的版本：

```c++
template <typename T> void move_middle_to_first(T result, T a, T b, T c) {
  if (*a < *b) {
    if (*b < *c) {
      std::swap(*result, *b);
    } else if (*a < *c) {
      std::swap(*result, *c);
    } else {
      std::swap(*result, *a);
    }
  } else {
    if (*a < *c) {
      std::swap(*result, *a);
    } else if (*b < *c) {
      std::swap(*result, *c);
    } else {
      std::swap(*result, *b);
    }
  }
}

// [first, last)
template <typename T> T Partition(T first, T last) {
  --last;
  move_middle_to_first(first, first, first + (last - first) / 2, last);
  T pivot = first;

  while (first < last) {
    while (first < last && *pivot <= *last)
      --last;

    while (first < last && *first <= *pivot)
      ++first;
    
    if (first < last) {
      std::swap(*first, *last);
    }

  }

  std::swap(*pivot, *first);

  return first;
}


template <typename T> void QuickSort(T first, T last) {
  if (first >= last)
    return;

  T pi = Partition(first, last);

  QuickSort(first, pi);
  QuickSort(pi + 1, last);
}
```



#### 延深点

以上只是讲述自己对快排的理解，相比于c++标准库提供的std::sort而言，我们自定义实现的快排程序，比起快排还要弱一些，主要是有一点没有考虑：当数据量比较小的时候，快排比插排还要慢一点，这是因为递归调用和元素腾挪的时间消耗，是有一个阈值的，所以c++标准库提供的sort函数在阈值小于一定数值的时候，会采用插入排序。
