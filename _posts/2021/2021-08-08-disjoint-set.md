---
layout:   post
title:   "并查集"
subtitle:  "C++ template rules"
date:    2021-08-08 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

### 什么是并查集

并查集，英文名为disjoint set或者union find，它是一种数据结构，主要用于解决图的一些连通问题，先来看一下它的定义：

![](/img/in-post/zhouhaijun-pic/disjoint-set.png)

首先它的内部有一系列的集合构成，对外提供的操作主要有：查找元素所在集合、合并两个集合。

它与树的结构是相反的，节点指向的都是自己的父亲节点。

### 并查集的应用场景

并查集要解决的问题场景都比较单一，比如找朋友问题等，实际工程里边遇到的一个经典的应用场景就是人脸聚类(leetcode上的这位网友写的蛮好的，我借鉴一下，原帖的链接在https://leetcode-cn.com/problems/number-of-provinces/solution/tu-jie-bing-cha-ji-by-time-limit-6x7p/)

也就是说，在现实场景下，同一个人在相同或不同抓拍摄像机下的抓拍图片会略有差别，比如下图，我们人类是可以识别出是同一个人，但是深度学习训练出来的算法未必能识别正确，算法是通过计算特征向量相似度来判别是否属于同一个人，同一个人的不同抓拍角度，它们的特征在计算相似度时就可能差异很大。

![](/img/in-post/zhouhaijun-pic/disjoint-set-app1.png)

还是以上面的图为例，我们依次对上面的五张图编号为a、b、c、d、e，如果直接拿a的特征与e的特征比对，得到的相似度可能很低，但是如果换一个思路：

a与b、c比对，三个特征计算的相似度为同一人；

c与d比对，相似度为同一人；

d与e比对，相似度为同一人；

其实上面的问题就转换为连通图问题：a、b、c为同一个集合，c与d为同一个集合，d又与e为同一个集合，这样的三个结合经过合并，最终就是同一个结合，这就是经典的并查集的操作。

### 并查集的操作

- 合并两个集合

![](/img/in-post/zhouhaijun-pic/disjoint-set-merge.png)

合并两个元素或节点时，就是各自向上查找两个节点所属的祖先，如果为同一个祖先，说明两个节点在同一个集合里，否则就将其中一个祖先替换为另外一个节点的祖先。

- 查找元素

![](/img/in-post/zhouhaijun-pic/disjoint-set-find.png)

查找元素，基本上就是沿着路径向上查找根节点，当然这里有一个小的优化点，就是每次查找时做一次路径压缩，这样提升合并或后续查找的速度，比如判断两个元素是不是在同一个集合。

### 代码模板

```c++
class UnionFind {
public:
    UnionFind(int n) {
        for (int i = 0; i < n; ++i) {
            parent.push_back(i);
        }
        count = n;
    }

    int find(int i) {
        int root = i;
        //  find root parent
        while (parent[root] != root) {
            root = parent[root];
        }

        // path compression
        while (parent[i] != i) {
            int temp = i;
            i = parent[i];
            parent[temp] = root;
        }

        return root;
    }

    void merge(int x, int y) {
        int xRep = find(x);
        int yRep = find(y);
        if (xRep != yRep) {
            parent[xRep] = yRep;
            -- count;
        }
        return;
    }

    int getCount() {
        return count;
    }
private:
    vector<int> parent;
    int count {0};
};
```



### 总结

并查集的应用场景比较单一，实际工程里边用到的这类算法有找朋友或人类聚类等，代码模板相对比较简单，这里记录一下，方便后续查找。
