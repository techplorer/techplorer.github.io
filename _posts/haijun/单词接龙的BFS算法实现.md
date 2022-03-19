### 题目

leetcode有一道单词接龙的题目，非常有意思，正好完美解释什么是BFS算法，原题是如下：

> 字典 wordList 中从单词 beginWord 和 endWord 的 转换序列 是一个按下述规格形成的序列：
>
> 
>
> 序列中第一个单词是 beginWord 。
>
> 序列中最后一个单词是 endWord 。
>
> 每次转换只能改变一个字母。
>
> 转换过程中的中间单词必须是字典 wordList 中的单词。

> 给你两个单词 beginWord 和 endWord 和一个字典 wordList ，找到从 beginWord 到 endWord 的 最短转换序列 中的 单词数目 。如果不存在这样的转换序列，返回 0。

> 

> 示例 1：

> 

> 输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]

> 输出：5

> 解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。

> 

> 来源：力扣（LeetCode）

> 链接：https://leetcode-cn.com/problems/word-ladder

> 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



这道题属于最短路径类题目，一般这类题目通常会想到dp和bfs，在这里的话，我们就考虑bfs。

之前我在《深度与广度优先搜索代码模板》这篇文章里贴过bfs的代码模板，其实这道题是完全可以用模板套用的。

按照题目的理解，我们可以把每个单词看成一个节点，那么所有节点之间的路径就组成了一个无向图，像下面这样(图摘自leetcode)：

![dan-ci-jie-long](images/dan-ci-jie-long.png)

而我们的任务就是在这个无向图中找到从起点到终点的最短距离，所以本质上这是一道无向图的bfs搜索题，而题目的难点就在于如何找到一个节点对应的其他节点，也就是类似下面的代码：

```c++
        for (int i = 0;i < node->children.size(); ++i) {
            todo.push(node->children[i]);
        }
```

其实想到了就挺简单的，我们的节点是一个字符串，那么只要对字符串中的每一个字符做'a'-'z'的穷举，满足条件即可认为是有效的节点，把这点相通了，代码就好写了。



### 求解方法一(bfs)

代码里增加注释了，可直接阅读源码。

```c++
class Solution {
public:
    int dfs(string& beginWord, string& endWord, vector<string>& wordList) {
        // 为加快检索速度，这里用哈希表替换vector
        unordered_set<string> dict(wordList.begin(), wordList.end());

        // 异常分支处理，如果不存在这样的解，则直接返回0
        if (dict.find(endWord) == dict.end()) {
            return 0;
        }

        queue<string> todo; // 记录待访问的节点
        unordered_set<string> visited; // 标记数组，用于存储已经遍历过的节点
        todo.push(beginWord);
        int ladder = 0; // 记录求解的值

        while (!todo.empty()) {
            int QLevelLen = todo.size();// 记录这一层的所有节点数，然后挨个遍历节点
            ++ ladder;
            for (int i = 0;i < QLevelLen; ++i) {
                string node = todo.front();
                todo.pop();

                if (node == endWord) {
                    return ladder; //找到了最后一个节点，返回计算结果
                }

                if (visited.find(node) != visited.end()) {
                    continue; // 已经访问这个节点，跳过
                }
                visited.insert(node); // 对已经访问的节点做一下标记
                
                // 寻找相邻的其他节点，也就是找到下一层的所有节点，并放到队列里
                for (int i = 0;i < node.size(); ++i) {
                    char ch = node[i];
                    for (int k = 0; k < 26; ++k) {
                        node[i] = 'a' + k;

                        // 如果在dict里查找到对应元素，表明找到了要遍历的下一个节点，放到队列里
                        if (dict.find(node) != dict.end()) {
                            todo.push(node);
                        }
                    }
                    node[i] = ch;
                }
            }

        }
        // 遍历所有节点，没有找到解，返回0，其实这里的代码不会执行的，因为在函数头已经做过异常处理
        return 0;
    }

    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        return dfs(beginWord, endWord, wordList);
    }
};
```



### 求解方法二(双向bfs)

上述的bfs本质上是一种暴力搜索，那其实还有一种优化的方案，即双向bfs：

```c++
class Solution {
public:
    int dfs(string& beginWord, string& endWord, vector<string>& wordList) {
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (dict.find(endWord) == dict.end()) {
            return 0;
        }
        queue<string> head;
        queue<string> tail;
        unordered_set<string> headVisited;
        unordered_set<string> tailVisited;
        head.push(beginWord);
        tail.push(endWord);

        int ladder = 0;

        while (!head.empty() && !tail.empty()) {
            // 每次遍历，都从head和tail中挑选元素少的队列，这样可以可能尽可能的少计算
            if (head.size() > tail.size()) {
                swap(head, tail);
                swap(headVisited, tailVisited);
            }

            int QLevelLen = head.size();
            for (int i = 0; i < QLevelLen; ++i) {
                string node = head.front();
                head.pop();

                // 表示两个队列的访问在该节点相遇，得到了问题的解
                if (tailVisited.find(node) != tailVisited.end()) {
                    return ladder;
                }

                if (headVisited.find(node) != headVisited.end()) {
                    continue;
                }
                headVisited.insert(node);

                for (int j = 0; j < node.size(); ++j) {
                    char ch = node[j];
                    for (int k = 0; k < 26; ++k) {
                        node[j] = 'a' + k;
                        if (dict.find(node) != dict.end()) {
                            head.push(node);
                        }
                    }
                    node[j] = ch;
                }
            }
            // 这里为什么要把ladder加一的操作放在这里呢？是因为出口在上面两队列相遇的地方
            ++ ladder;
        }
        return 0;
    }

    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        return dfs(beginWord, endWord, wordList);
    }
};
```



为什么双向bfs会比单向bfs快呢？这里用网友的一张图即可说明：

![double-bfs](images/double-bfs.png)

因为双向bfs的搜索空间比单向bfs的小，也就意味者我们在访问时速度会更快一点，但是有一点需要注意一下，如果我们能确定从起点到终点一定有解的话，那么双向bfs会更快一点，否则双向bfs就变成单向bfs了。

### 双向bfs代码模板

经过上述分析，我们基本可以得出双向bfs的代码模板了：

```c++
void bfs(Node* start, Node* end) {
    if (!start || !end) {
        return;
    }

    queue<Node*> head;
    queue<Node*> tail;
    unordered_map<int,int> headVisited;
    unordered_map<int,int> tailVisited;
    head.push(start);
    tail.push(end);

    while (!head.empty() && !tail.empty()) {
        //  make sure head is the shortest size queue
        if (head.size() > tail.size()) {
            swap(head, tail);
            swap(headVisited, tailVisited);
        }

        int QLevelLen = head.size();
        for (int i = 0;i < QLevelLen; ++i) {
            
            Node* node = head.front();
            head.pop();

            if (tailVisited.find(node) != tailVisited.end()) {
                //  find solution
                return;
            }

            if (headVisited.find(node->val) != headVisited.end()) {
                continue; // already visited, skip the node
            }
            headVisited[node->val] = 1; // mark the node after being visited
            
            for (int i = 0;i < node->children.size(); ++i) {
                todo.push(node->children[i]); // add the next visited nodes
            }
        }
    }

}
```



### 总结

单词接龙这道题我刷了不下5遍，因为我觉得这道题是能看到bfs算法本质的一道题，多刷几遍，确实会加深理解。