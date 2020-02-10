+++
title = "[DP][NOIP2006] 能量项链"
weight = 1
order = 1
date = 2020-02-04
#insert_anchor_links = "right"

[taxonomies]
tags = ["solutions"]
+++

[题目](https://vjudge.net/problem/HRBUST-1376)
[题目](https://vijos.org/p/1312)

这题怎么这么像武大算法用的教材上矩阵相乘的例题

诶，还真的是一样的，可惜就是有环，环的问题可以参考[这里](https://oi-wiki.org/dp/interval/)

> 我们将这条链延长两倍，变成$2n$堆，其中第$i$堆与第$n+i$堆相同，用动态规划求解后，取$f(1,n),f(2,n+1),...,f(i,n+i-1)$中的最优值，即为最后的答案。

第$i$个珠子的头标记为$w(i)$，
尾标记为$w(i+1)$

先无视环，当成串处理。

令$f(i,j)$为合并$[i, j)$之间的珠子所取得的最优解，
则状态转移方程：
$$
f(i, j) = max_{\forall k \in [i+1, j)} f(i, k) + f(k, j) + w(i)w(k)w(j)
$$
注意$k$要从$i+1$开始，要不是就死循环了。

采用上述环的处理方法后，
解为从某个珠子处展开成串后的最大者，
即$max_{\forall i \in [0, n)} f(i, n+i)$

关于计算顺序，就偷懒用记忆化递归了。

```python
INVALID = -1

def main():
    n = int(input().strip())
    w = list(map(int, input().strip().split(' ')))

    w.extend(w)
    w.append(w[0])
    dp = [[INVALID for j in range(len(w))] for i in range(len(w))]
    for i in range(1, len(w)):
        dp[i-1][i] = 0
    # print(w, dp)

    def f(i, j):
        if dp[i][j] != INVALID:
            return dp[i][j]
        m = INVALID
        for k in range(i + 1, j):
            m = max(m, f(i, k) + f(k, j) + w[i]*w[k]*w[j])
        dp[i][j] = m
        return m

    res = INVALID
    for i in range(n):
        res = max(res, f(i, i + n))
    print(res)


if __name__ == "__main__":
    while True:
        try:
            main()
        except EOFError:
            break

```