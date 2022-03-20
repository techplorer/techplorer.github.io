---
layout:     post
title:      "Trie的算法实现"
subtitle:   "Trie algorithm "
date:       2021-04-05 12:00:00
author:     "zhouhaijun"
catalog: false
header-style: text
tags:
  - 算法
---

### 什么是Trie

Trie，也叫prefix tree，即前缀树或者字典树，一般是用于字符查找或前缀字符查找，我们工作当中经常也能接触到这种数据结构，举几个例子来说：

- 自动补齐

在搜索引擎里搜索字符时，会自动补齐字符：

![](/img/in-post/zhouhaijun-pic/trie-example-1-8d62fb3b7c5a4114a15ee198db9bb689.png)

再比如，我们用IDE写代码时，工具会自动提示我们要补齐的代码：

![](/img/in-post/zhouhaijun-pic/trie-example-2-53b3c3ade1c541c48fbaaa0e1b03d22e.png)

- 拼写错误检查

使用word文档拼错单词会提示错误，还有在IDE拼错单词也会提示你错误：

![trie-example-3](http://www.techplorer.cn/upload/2021/05/trie-example-3-e4df78c1140543d8bacfddacf8bc792a.png)

- 路由

在一些路由算法里也会用到Trie的数据结构，比如同时存在两个下面两个路由：

| IP               | Router |
| ---------------- | ------ |
| 192.168.20.16/28 | A      |
| 192.168.0.0/16   | B      |

假如有一个IP192.168.20.19，那么在路由选择时，一般会选择前缀最长的那个路由，也就是A。



### 数据结构

以上都是一些经典的Trie的算法应用，那么它在计算机内部的存储是怎么实现的呢？

下面一幅图能生动直观的表示Trie的内部数据结构：

![trie](http://www.techplorer.cn/upload/2021/05/trie-65c02977160e4036869e395d0ff21e01.png)

这里要注意的一点是，Trie跟二叉搜索树不一样，它的每个节点存储的并非是整个单词或字符串，而是单个字符，从第一个节点按照深度优先的策略依次往下遍历，遍历完成后的路径就才是一个完整的字符串。

### 模板代码

- 描述

> Trie（发音类似 "try"）或者说 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。
>
> 请你实现 Trie 类：
>
> Trie() 初始化前缀树对象。
> void insert(String word) 向前缀树中插入字符串 word 。
> boolean search(String word) 如果字符串 word 在前缀树中，返回 true（即，在检索之前已经插入）；否则，返回 false 。
> boolean startsWith(String prefix) 如果之前已经插入的字符串 word 的前缀之一为 prefix ，返回 true ；否则，返回 false 。
>
> 来源：力扣（LeetCode）
> 链接：https://leetcode-cn.com/problems/implement-trie-prefix-tree
> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



- 代码实现

```c++
class Trie {

struct TrieNode {
    unordered_map<char, TrieNode*> childTable;
    bool isEnd;

    TrieNode() : isEnd(false) {}
};

public:
    Trie() {
        root = new TrieNode();
    }

    bool search(const string& word) {
        return find(word, true);
    }

    bool startsWith(const string& prefix) {
        return find(prefix, false);
    }

    void insert(const string& word) {
        TrieNode* curr = root;
        for (int i =0;i < word.size(); ++i) {
            auto search = curr->childTable.find(word[i]);
            if (search == curr->childTable.end()) {
                curr->childTable[word[i]] = new TrieNode;
            }
            curr = curr->childTable[word[i]];
        }
        curr->isEnd = true;
    }

private:
    bool find(const string& word, bool exact_match) {
        TrieNode* curr = root;
        for (int i = 0;i < word.size(); ++i) {
            auto search = curr->childTable.find(word[i]);
            if (search == curr->childTable.end()) {
                return false;
            }
            curr = curr->childTable[word[i]];
        }

        if (exact_match) {
            return curr->isEnd ? true : false;
        }

        return true;
    } 

    TrieNode* root;

};
```



### 总结

Trie是一种日常工作和学习中经常能接触到的一种数据结构，它的算法实现也比较简单，设计的原理基本也是按照空间换时间的一种思路。
