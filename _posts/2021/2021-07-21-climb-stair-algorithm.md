---
layout:   post
title:   "爬楼梯问题的几种算法实现"
subtitle:  ""
date:    2021-05-11 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 算法
---

### 爬楼梯

这是一道来自leetcode上的题，题目的描述是这样的：

> 假设你正在爬楼梯。需要 *n* 阶你才能到达楼顶。
>
> 每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> **注意：**给定 *n* 是一个正整数。



### 暴力求解

站在第0阶楼梯时，这时面临的选择有两个：向上走一步到达第一阶或者向上走两步到达第二阶，当走到第2阶楼梯时，又同样面临一摸一样的选择，依次类推下去，直到走到第N阶台阶。

按照上面的思路，我们使用dfs可以写出如下代码：

```c++
class Solution {
public:
    void helper(int from, int end, int& ans) {
        if (from == end) {
            // find a solution
            ++ ans;
            return;
        }

        if (from > end) {
            return; // filter invalid solution
        }

        helper(from + 1, end, ans); // move one way upward
        helper(from + 2, end, ans); // move two ways upward
        
    }

    int climbStairs(int n) {
        int ans = 0;
        helper(0, n, ans);
        return ans;
    }
};
```



分析一下这个程序的复杂度，因为每往上走楼梯时，都会面临2个选择：要么向上走一步要么向上走两步，所以其时间复杂度是o(2^n)，属于指数级的复杂度，当然用上面的代码在leetcode跑肯定会超时。

### 递归

此时我们换一种思路思考：因为每次要么走1步要么走2步，所以如果要走到第N阶台阶的话，要么是从第N-1级台阶走上来的，要么是从第N-2阶台阶走上来的，只有这两种情况。那么我们如果记f(N)为从第一阶走到第N阶的总的走法，那么就可以得到如下的关系表达式：
$$
f(N) = f(N - 1) + f(N - 2)
$$
也就是说如果我们能计算出f(N - 1)和f(N - 2)的话，f(N)也就解出来了，而如果要求解f(N - 1)的话，我们只需要求解f(N - 2)和f(N - 3)，以此类推，最终我们会推算到f(1)和f(2)，通过题目，我们可以很明显地得到这两个值：
$$
f(1)=1, f(2) = 2
$$
有了上述的分析和思考，我们可以将程序改写为如下：

```c++
class Solution {
public:
    int climbStairs(int n) {
        // terminator
        if (n == 1 || n == 2) {
            return n;
        }

        // drill down
        return climbStairs(n - 1) + climbStairs(n - 2);
    }
};
```

代码本身是非常简洁的，但是分析上述代码的复杂度时，我们发现其时间复杂度也是o(2^n)，参见下图(爬楼梯问题本质上和斐波那契数列是一样的，所以我从网上摘了一个斐波那契数列计算的状态树)

![](/img/in-post/zhouhaijun-pic/recursion_status_tree.png)

虽然没有改变算法的时间复杂度，但是上述思考问题的方式确实很有意思，是一种分而治之的思想，我们接着往下优化。



### 记忆化递归

上面的递归程序时间复杂度是指数级，相对来说复杂度很高，那么通过上面的程序计算的状态树，可以发现有大量的重复计算，比如如果要计算f(7)，那么f(5)就要重复计算两次，f(4)要重复计算3次，所以我们很自然地想到如果把这些计算的中间结果都缓存起来，就可以避免大量的重复计算，从而提升算法的性能。

优化后的代码就变成下面这样：

```c++
class Solution {
public:
    int helper(int n, vector<int>& memo) {
        if (n == 1 || n == 2) {
            return n; // 已经到了最底层了，memo[0] memo[1] memo[3]没有必要再存储了
        }

        if (memo[n] != INT_MIN) {
            return memo[n]; // already computed
        }

        memo[n] = helper(n - 1, memo) + helper(n -2, memo);
        return memo[n];
    }

    int climbStairs(int n) {
        vector<int> memo(n + 1, INT_MIN);
        return helper(n, memo);
    }
};
```



相比之前的两种解法，带有记忆的递归算法在时间复杂度上是o(n)，时间复杂度一下从指数级降低为线性时间，性能上有大幅度的提升。

同样的leetcode上用这种算法也通过了:

![](/img/in-post/zhouhaijun-pic/climb_stairs_recursion_with_memo.png)

### 动态规划

上面也说了，递归求解的这种思路的确很特别，因为它很适合人脑的思考方式：先将一个大问题能分解成若干子问题，再对子问题进行求解，当求解完子问题，那么大问题也就迎刃而解了，而在求解子问题时，子问题又可以拆解为子子问题，这样依次递归下去，直到碰到终止条件，之后再一步步的返回。

其实对于计算机而言，还有一种方式，那就是循环，使用循环它可以达到同样的效果：先将相关的子问题求解，再利用递推关系，将状态转移到后面的一个问题，依次递推下去，直到碰到最后的大问题。

所以两者的区别在于解决问题的视角不一样：递归用的是top-down，而循环用的是bottom-up。

回到问题本身，如果要用动态规划，这道题可以这么解：



```c++
class Solution {
public:
    int climbStairs(int n) {
        vector<int> dp(n + 1, INT_MIN);
        dp[1] = 1;
        dp[2] = 2;

        for (int i = 3; i <= n; ++i) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }
};
```

因为每个数只计算了一次，所以动态规划的时间复杂度也是o(n)，其实不论是上述的记忆化递归算法还是动态规划，都已经算是不错的结果，但是我们进一步思考，会发现从空间上还可以进一步的优化：因为两者都是通过一个类似数组的这么一个数据结构，将中间所有的计算结果都缓存了，但是实际上我们要求解f(N)，只依赖了f(N - 1)和f(N - 2)，所以没有必要把整个计算过程的中间结果都缓存，每次缓存前面两个子问题的计算结果，再顺序递推下去即可，于是进一步的代码优化：

```c++
class Solution {
public:
    int climbStairs(int n) {
        if (n == 1 || n == 2) {
            return n;
        }

        int one_step = 2;
        int two_steps = 1;
        int all_ways = 0; 

        for (int i = 3; i <= n; ++ i) {
            all_ways = one_step + two_steps;
            two_steps = one_step;
            one_step = all_ways;
        }

        return all_ways;
    }
};
```

从leetcode的测试用来，空间上有一定的优化，但是优化的不明显，其实也跟测试用例的规模有关:

![](/img/in-post/zhouhaijun-pic/climb_stairs_dp.png)

### 总结

本文以爬楼梯问题为例子，依次从暴力求解、递归、记忆化递归、动态规划的视角分别做了代码和算法复杂度的分析，其实除了爬楼梯，斐波那契数列以及零钱兑换都是类似的思想。

那么在文章的末尾留下一个小的尾巴：通过上述文章的分析，我们发现无论是记忆化递归还是动态规划，两者在时间复杂度上都是o(n)，那么两者的区别在哪呢？或者说在实际的算法题中，我们在二者之间应该如何择优选择呢？
