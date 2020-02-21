+++
title = "Fenwick Tree"
weight = 1
order = 1
date = 2020-02-04
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]

+++

[A nice post about Fenwick tree on topcoder](https://www.topcoder.com/community/competitive-programming/tutorials/binary-indexed-trees/)

<!--
想要/存放的值：

| $i$       | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  | 14  | 15  | 16  |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| $arr[i]$  | 1   | 0   | 2   | 1   | 1   | 3   | 0   | 4   | 2   | 5   | 2   | 2   | 3   | 1   | 0   | 2   |
| $sum[i]$  | 1   | 1   | 3   | 4   | 5   | 8   | 8   | 12  | 14  | 19  | 21  | 23  | 26  | 27  | 27  | 29  |
| $tree[i]$ | 1   | 1   | 2   | 4   | 1   | 4   | 0   | 12  | 2   | 7   | 2   | 11  | 3   | 4   | 0   | 29  |

数组每个格子负责的范围：


| $i$    | 1   | 2    | 3   | 4    | 5   | 6    | 7   | 8    | 9   | 10    | 11  | 12    | 13  | 14     | 15  | 16    |
| ------ | --- | ---- | --- | ---- | --- | ---- | --- | ---- | --- | ----- | --- | ----- | --- | ------ | --- | ----- |
| $tree$ | 1   | 1..2 | 3   | 1..4 | 5   | 5..6 | 7   | 1..8 | 9   | 9..10 | 11  | 9..12 | 13  | 13..14 | 15  | 1..16 |

画成图

![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200204233119.png)
![](https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200204233432.png)
-->

- `lowbit(x)`
  - `x`二进制表示中最低的`1`位
  - 令`x`表示为`a1b`
  - `b`全`0`
  - `a`有`0`有`1`
  - `-x = ~(a1b) + 1 = (~a)0(~b) + 1 = (~a)1b`
  - `-x & x = (~a)1b & a1b = (0..0)1(0..0)`

- `sum(idx)`
  - 读取前缀和
  ```c++
  int sum(int idx) {
      int sum = 0;
      while (idx > 0) {
          sum += tree[idx];
          idx -= (idx & -idx);
      }
      return sum;
  }
  ```

- `update(idx, delta)`
  - 更新某位置的值
  ```c++
   void update(int idx, int delta) {
      while (idx <= MaxIdx) {
          tree[idx] += delta;
          idx += (idx & -idx);
      }
  } 
  ```

- `at(idx)`
  - 读取某位置的值
  - 如果想$O(1)$的话就乖乖再存一遍数组
  - naïve的方法就直接返回`sum(idx) - sum(idx - 1)`
  - 注意到在计算`sum(idx)`和`sum(idx - 1)`时，在树中的路径最后总是会相交
  - 只需找到交点，计算相交之前的差值即可
  - 取`y = x - 1`，若`y = a0b`则`x = a1(~b)`
  - 每次迭代从`x`中减去`lowbit(x)`， 即`x`替换为`z = a0(~b)`
  - 注意到前面的`a0`都是相同的，算法每次从`b`中丢掉一位，最后总会相等
  - 最差$O(logn)$，即当`x == n`时
  - 当`x`为奇数时复杂度$O(1)$
  ```c++
  int at(int x){
      int sum = tree[x];
      if (x > 0) {
          int z = x - (x & -x);
          int y = x - 1;
          // at some iteration y will become z
          while (y != z) {
              sum -= tree[y]; 
              y -= (y & -y);
          }
      }
      return sum;
  }
  ```

- `scale(c)`
  - 将整棵树/数组缩放常数倍
  - $O(logn)$
  ```c++
  void scale(int c){
      for (int i = 1; i <= MaxIdx; i++) {
          tree[i] = c > 0 ? tree[i] * c: tree[i] / (-c);
      }
  }
  ```

- `find(sum)`
  - 根据前缀和找下标
  - `sum(idx)`的逆操作
  - 原理是根据二进制位从高到底来进行二叉搜索
  - 如果有多个下标对应该前缀和则返回其中最大的一个
  - $O(logn)$
  ```c++
  int find(int sum) {
      int idx = 0;
      // initialy mask is the greatest bit of MaxIdx
      // mask means current interval
      while (mask != 0) {
          // midpoint of the interval
          int t = idx + mask;
          mask >>= 1;
          if (t > MaxIdx) {
              continue;
          }
          if (sum >= tree[t]) {
              // if the current cumulative frequency is equal to sum
              // we are still looking for a higher index (if exists)
              idx = t;
              sum -= tree[t];
          }
      }
      if (sum != 0) {
          return -1;
      } else {
          return idx;
      }
  }
  ```