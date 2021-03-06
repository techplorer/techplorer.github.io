---
layout:   post
title:   "快速排序、归并排序、堆排序"
subtitle:  ""
date:    2021-09-03 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
 - 排序
---

### 快速排序

- 动画

![](/img/in-post/zhouhaijun-pic/quick-sort-demo.gif)



- 代码模板

```c++
// 找到一个基准值，让所有左边的元素小于它， 且右边的元素大于它
int partition(vector<int>& nums, int left, int right) {
    // 随机挑选一个pivot，尽量避免出现o(n^2)的复杂度
    int random_pivot_index = rand() %(right - left + 1) + left;
    swap(nums[random_pivot_index], nums[right]);
    int pivot = right;

    while (true) {
        // 从左到右扫描，直到遇到一个大于pivot的元素，下面的这行代码很关键，它可以确保最终的返回条件，即right是一定大于或等于pivot
        while (left < right && nums[left] <= nums[pivot]) ++left;

        // 从右向左扫描，直到遇到一个小于pivot的元素
        while (left < right && nums[right] >= nums[pivot]) --right;
        if (left < right) {
            // 满足交换条件，则进行交换
            swap(nums[left], nums[right]);
        } else {
            break;
        }
    }

    // right指向的元素或节点，就是pivot应该放的位置，所以把两者进行交换
    swap(nums[pivot], nums[right]);
    return right;
}

void quickSort(vector<int>& nums, int left, int right) {
    if (left >= right) return;
    int pi = partition(nums, left, right);
    quickSort(nums, left, pi - 1);
    quickSort(nums, pi + 1, right);
}
```



- 算法复杂度

时间复杂度平均是o(n*logn)，空间复杂度是o(1)



### 归并排序

- 动画
![](/img/in-post/zhouhaijun-pic/merger-sort-demo-e8d106f11e2f4567afaf4a24e4e9cbc7.gif)

- 代码

```c++
void merge(vector<int>& nums, int left, int mid, int right) {
    vector<int> tmp(right - left + 1);

    // 左边排序好的数组
    int leftArrMin = left;
    int leftArrMax = mid;
    int leftArrIdx = leftArrMin;

    // 右边已经排序好的数组
    int rightArrMin = mid + 1;
    int rightArrMax = right;
    int rightArrIdx = rightArrMin;

    int tmpIdx = 0;

    // 从两个已经排序好的数组，优先把较小的元素放到临时数组tmp里
    while (leftArrIdx <= leftArrMax && rightArrIdx <= rightArrMax) {
        tmp[tmpIdx++] = nums[leftArrIdx] < nums[rightArrIdx] ? nums[leftArrIdx++] : nums[rightArrIdx++];
    }

    // 如果发现左边的数组没有填充完，则把剩下的元素依次填充到临时数组tmp里 
    while (leftArrIdx <= leftArrMax) {
        tmp[tmpIdx++] = nums[leftArrIdx++];
    }

    // 如果发现右边的数组没有填充完，则把剩下的元素依次填充到临时数组tmp里
    while (rightArrIdx <= rightArrMax) {
        tmp[tmpIdx++] = nums[rightArrIdx++];
    }

    // 最后一步，将归并好的临时数组tmp的所有元素依次拷贝到原数组中
    for (tmpIdx = 0, leftArrIdx = left; tmpIdx < tmp.size(); ) {
        nums[leftArrIdx++] = tmp[tmpIdx++];
    }
}



void mergeSort(vector<int>& nums, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(nums, left, mid); // 左边的数组进行排序
    mergeSort(nums, mid + 1, right); // 对右边的数组进行排序
    merge(nums, left, mid, right); // 将上述已经排好序的数组进行合并
}
```

- 算法复杂度

时间复杂度平均是o(n*logn)，空间复杂度是o(n)

### 堆排序

- 动画

![heap-sort-demo](http://www.techplorer.cn/upload/2021/06/heap-sort-demo-c598abb036c64d8cb0f314fc780d4133.gif)

- 代码

```c++
// 从i到max做一次排序
void heapify(vector<int>& nums, int i, int max) {
    
    while (i < max) {
        int largest = i;
        int leftChild = 2 * i + 1;
        int rightChild = 2 * i + 2;

        if (leftChild < max && nums[largest] < nums[leftChild]) {
            largest = leftChild; 
        }

        if (rightChild < max && nums[largest] < nums[rightChild]) {
            largest = rightChild;
        }

        // 此时说明从i到max的所有节点已经满足大顶堆，可以退出
        if (largest == i) {
            break;
        }

        // 走到这里，说明左孩子或者有孩子比父亲节点大，这时需要继续往下遍历，所有需要交换，同时更新i
        swap(nums[largest], nums[i]);
        i = largest;
    }
}

void buildMaxHeap(vector<int>& nums) {
    // 自底部向上，先找到最后一个parent
    int parent = nums.size() / 2 - 1;

    while (parent >= 0) {
        heapify(nums, parent, nums.size());

        // 把排好序的节点往上提
        parent -= 1;
    }
}


void heapSort(vector<int>& nums) {
    // 建立大顶堆
    buildMaxHeap(nums);

    int lastElement = nums.size() - 1;

    while (lastElement > 0) {
        // nums[0]是最大的元素，所以每次把nums[0]与最后待排序的元素交互
        swap(nums[0], nums[lastElement]);

        // 上面交换完后，大顶堆的结构可能破坏了，这时需要基于nums[0]再向下做一次heapify
        heapify(nums, 0, lastElement);

        // 继续开始准备下一个待排序的元素
        lastElement -= 1;
    }
}
```

- 算法复杂度

时间复杂度平均是o(n*logn)，空间复杂度是o(1)



### 总结

本文总结了几种经典排序算法的运行过程及模板代码，也是工业上常用的三种算法，平均时间复杂度均为o(n*logn)。
