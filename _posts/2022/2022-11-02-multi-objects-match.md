---
layout:   post
title:   "多目标匹配(匈牙利算法)"
subtitle:  ""
date:    2022-11-02 22:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

### 背景

二分图的最大匹配问题，在日常生活中也很常见，比如外卖场景，订单与骑手的匹配，滴滴网约车司机与乘客的匹配，都会用到二分图匹配算法，结合自己的思考聊聊二分图匹配问题。



### 无向图最大边匹配

为了便于描述，先来放一个无向二分图：

- 我们先假设有两个不相交的集合S,S中含元素A、B、C，另一集合D，D中含元素a、b、c；

- S和D的元素之间的关系如上述图所示;

  ![无向图](/img/in-post/zhouhaijun-pic/2022/multi-objects-match-1.jpeg)

如上图所示，我们要找出从S到D的最大匹配，该怎么实现呢？

简单分析一下：

因为是无向图，没有权重，所以我们要想让S和D实现最大的匹配，问题进一步转化为找图中的最大匹配边，不难看出，在这种场景下，核心的问题就在于如何解决目标匹配冲突，比如A和B同时可以选择a，那么到底选择的匹配边是A-a还是B-a。

有一种思路是这样的：当发生目标匹配冲突时，比如A-a是已经匹配的边，此时B发现a也可以匹配，这时试图让A去找其他的目标，发现A可以与c匹配，那么匹配成功：B-a匹配，A-c匹配。A试图找其他目标和B找目标都是同样的过程，可以用计算机的递归来解决，分析到这里，整个问题就迎刃而解。

### 匈牙利算法

上述这个过程，其实就是匈牙利算法，比较粗糙，再来一个专业一点的描述：

先定一个术语：增广路径：我们从集合S出发，去匹配集合D中的元素，按照未匹配的边->匹配的边->未匹配的边->....->未匹配的边，依次交替进行，最终形成一条路径，叫增广路径。

增广路中的匹配边就是目标有冲突匹配的地方，解决的办法就是让发生冲突的顶点去找其他可能存在的匹配目标，所以整个匹配的过程，可以总结为如下过程：

遍历集合S中的每个元素，与集合D中的每个元素匹配，尝试去找一条增广路，如果找到，说明匹配成功，如果没找到，则匹配失败。

用图来描述一下过程：

![图1](/img/in-post/zhouhaijun-pic/2022/multi-objects-match-2.jpeg)

![图2](/img/in-post/zhouhaijun-pic/2022/multi-objects-match-3.jpeg)

![图3](/img/in-post/zhouhaijun-pic/2022/multi-objects-match-4.jpeg)



### 代码模板

结合代码，理解会更深刻一些：

```
#include <iostream>
#include <vector>

class Hungarian {

public:
  Hungarian(std::vector<std::vector<bool>> &g) {
    graph = std::move(g);

    maxLeftVertexNums = graph.size();
    maxRightVertexNums = graph[0].size();
    
    matchPairFromRight.reserve(maxRightVertexNums);
    for (int i = 0; i < maxRightVertexNums; ++i) {
      matchPairFromRight[i] = -1;
    }
    
    visitedRightVertex.reserve(maxRightVertexNums);

  }

  int MaxMatch() {
    int maxMatchEdges = 0;
    for (int i = 0; i < maxLeftVertexNums; ++i) {
      // 每一次match之前，要对visitedRightVertex做清零
      clearVisited();

      maxMatchEdges = find(i) ? (maxMatchEdges + 1) : maxMatchEdges;
    }
    
    return maxMatchEdges;

  }

  void print() {
    for (int i = 0; i < maxRightVertexNums; ++i) {
      std::cout << i << " -> " << matchPairFromRight[i] << std::endl;
    }
  }

private:
  void clearVisited() {
    for (int j = 0; j < maxRightVertexNums; ++j) {
      visitedRightVertex[j] = false;
    }
  }

  bool find(int srcVertexIdx) {
    for (int idx = 0; idx < maxRightVertexNums; ++idx) {
      if (graph[srcVertexIdx][idx] && !visitedRightVertex[idx]) {
        visitedRightVertex[idx] = true;

        if (matchPairFromRight[idx] == -1 || find(matchPairFromRight[idx])) {
          matchPairFromRight[idx] = srcVertexIdx;
          return true;
        }
      }
    }
    
    return false;

  }

private:
  int maxLeftVertexNums = 0;
  int maxRightVertexNums = 0;

  /*

   * 假定我们有两个不相交的集合A和B，分别用左集合和右集合来区分，匹配的时候从左边集合向右表集合搜索，
   * 所以visitedRightVertex表示左集合顶点在一次搜索中有没有被访问，避免出现死递归
     */
       std::vector<bool> visitedRightVertex;

  /*

   * 表示左边集合中顶点有没有匹配到右边的顶点，存放的是节点的索引，如果没有匹配存放-1
     */
       std::vector<int> matchPairFromRight;

  std::vector<std::vector<bool>> graph; // 邻接矩阵
};

int main(void) {
  std::vector<std::vector<bool>> graph;
  graph.push_back({1, 1, 0});
  graph.push_back({1, 0, 0});
  graph.push_back({0, 1, 0});

  Hungarian h(graph);
  int matchNums = h.MaxMatch();
  std::cout << "match num:" << matchNums << std::endl;
  h.print();

  return 0;
}
```

### 总结

匈牙利算法是解决二分图中的常用思路，但是我们发现其实这种算法没有考虑权重，即有向带权图，而我们生活中接触的问题都比较复杂，问题简化为有向带权图往往是一种更优的解决方案，下篇文章写一篇关于有向带权图的理解。
