---
layout:   post
title:   "二叉堆"
subtitle:  "heap"
date:    2021-07-15 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

- ### 堆

  ##### 定义

  堆是指在一堆数据里面能快速查找到最大值或最小值，这样的一种数据结构我们把它叫做堆。

  ##### 应用场景

  实际应用中，比较多的有两种：

  - 优先级队列

    比如我们在实际的工作中，可能遇到的场景：把任务放到内存数据，但是取任务时我们往往会选择最高优先级的任务，优先处理最高优先级到任务，这种就是优先级队列。

  - 堆排序

  ##### 性质

  简单点理解，可以把堆当作是一种接口，只要能满足以下几个特点的实现，都可以称之为堆：

  - 查找最大值(或最小值):  o(1)
  - 删除元素: o(logn)
  - 插入元素:  o(logn)或o(1)

  根据实际的应用场景，堆又分为大顶堆和小顶堆：父节点大于或等于孩子节点，我们叫大顶堆；父节点小于或等于孩子节点，我们叫做小顶推。
![](/img/in-post/zhouhaijun-pic/堆定义.png)


  ### 二叉堆

  上面讲了堆的定义，实际上堆的实现有很多，比如二叉堆、斐波拉契堆、严格的斐波拉契堆等等，其中二叉堆是一种实现上比较简单的一种数据结构。

  ##### 二叉堆定义

  不管是哪种堆的实现，最终都是要满足堆的性质，二叉堆是基于树的一种堆实现，它满足以下两个特点：

  - 完全二叉树

    完全二叉树，即除叶子节点之外的所有节点都是满的

    (这里放一张总结图，网友总结的蛮好的，我这里就直接拿过来用了)通过下面的图可以很生动的解释什么是完全二叉树，其实这里可以发现，二叉堆虽然是基于树结构的定义，但是在实现的时候我们往往用的是一维数组的数据结构。
![](/img/in-post/zhouhaijun-pic/完全二叉树.png)


  - 属性

    以下两个条件二选一：

    - 父节点大于等于孩子节点：大顶堆(max-heaps)
    - 父节点小于等于孩子节点：小顶堆(min-heaps)

  ##### 二叉堆时间复杂度

  不过有一点需要注意，在计算机的工业领域里，二叉堆并不是堆实现的最优算法，通过下面的这张图可以发现(参见下图-摘自维基百科)，二叉堆顶多只能算是入门级别。
![](/img/in-post/zhouhaijun-pic/堆时间复杂度.png)


  ##### 二叉堆的操作

  - 查找最大/小值

    查找最大值或最小值，其实就是访问根节点，也就是数组中的第一个元素，那么操作也是o(1)

  - 插入元素

    插入元素要做到log(n)，它是怎么实现的呢？还是以一个图来表示中间的过程：
![](/img/in-post/zhouhaijun-pic/Push-min-heap.png)

  这是一个小顶堆，即父节点小于等于左右两个孩子节点，现在要插入一个元素2，那么它的过程是这样：

  1. 先将元素2插到最后一个节点的后面；
  2. 元素2与父亲节点比较大小，若比父亲节点小，则交换，这里父亲节点是5，满足交换条件；
  3. 重复上述步骤，2与3也满足交换条件；
  4. 达到元素顶，终止；

  上面的整个过程也叫heapifyUp，那么在最坏的情况下，它从树的低端走到顶端，所以最差的情况下算法复杂度是log(n)。

  - 删除元素

    类似上面插入元素的过程，删除元素也差不多，只不过这次是反着来，见下图：
![](/img/in-post/zhouhaijun-pic/Pop-min-heap.png)


  假设我们要删除的元素就是最小值，它的过程是这样：

  1. 将最小值2取出并保存临时变量，取出的操作是o(1)；
  2. 取出末尾节点的元素18覆盖到最小值所在的位置上，并删除末尾节点；
  3. 将顶部元素18与孩子节点比较，若大于左右孩子的较小者，则交互，18大于3，满足交换条件；
  4. 重复上述步骤，18大于8，满足交换条件；
  5. 达到叶子节点，终止；

  上面的这个过程也叫heapifyDown，与上面删除元素的操作类似，只是从顶部往低端走，最坏的情况下时间复杂度是o(logn)。

  

  ##### 二叉堆代码实现

  [参见代码仓库](https://gitee.com/techplorer/leetcode/tree/master/code)  

  

  ### leetcode

  leetcode上可以试一试用二叉堆来解决实际的算法问题：

  - [最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)
  - [前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

  

  ### 资料

  - [https://en.wikipedia.org/wiki/Heap_(data_structure)](https://en.wikipedia.org/wiki/Heap_(data_structure))
  - [https://en.wikipedia.org/wiki/Binary_heap](https://en.wikipedia.org/wiki/Binary_heap)

  - [https://runestone.academy/runestone/books/published/pythonds/Trees/BinaryHeapImplementation.html](https://runestone.academy/runestone/books/published/pythonds/Trees/BinaryHeapImplementation.html)

