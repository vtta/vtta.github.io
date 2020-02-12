+++
title = "Rolling Hashes and Bloom Filters"
weight = 1
order = 1
date = 2020-02-11
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]

+++

## 二叉搜索的本质
存在一个函数`ok`可以将一个串的每个元素映射到布尔值, 
而且这个布尔向量可以被划分为不同的两块, 
每块要么全是`true`要么全是`false`, 
那么二叉搜索就是找到这两块的拐点. 

对于全是`true`或者全是`false`的情况, 
可以认为索引是`-1`或者`n`的位置存在一个哨兵, 
把这个哨兵当作不同的块.

## 例子 
`std::lower_bound(l, r, x)`

语义: 在$[l, r)$中找到第一个不小于`x`的位置, 若不存在则返回`r`

例子: 假设串是`[1, 3, 5, 7, 9, 11, 13]`, `x`是10
那么`ok`就是`lambda y: y < x`

```
index: -1 0 1 2 3 4  5  6  7
value: -∞ 1 3 5 7 9 11 13 +∞
ok?  :    T T T T T  F  F      x = 10
ok?  :  T F F F F F  F  F      x = -5
ok?  :    T T T T T  T  T  F   x = 15
```

## 模版写法
说找拐点, 那到底是返回拐点前一个后一个还是哪个下标呢?

常用的有四种: 

1. 第一个不满足条件的元素: T T T **F** F

    典例就是`std::lower_bound`, `ok`是`lambda y: y < x`

```c++
while (l < r) {
    auto m = l + (r - l) / 2;
    if (ok(m)) {
        l = m + 1;
    } else {
        r = m;
    }
}
return l;
```

2. 第一个满足条件的元素: F F F **T** T

    典例就是`std::upper_bound`, `ok`是`lambda y: y > x`

```c++
while (l < r) {
    auto m = l + (r - l) / 2;
    if (!ok(m)) {
        l = m + 1;
    } else {
        r = m;
    }
}
return l;
```

3. 最后一个满足条件的元素: T T **T** F F

```c++
auto p = l;
for (auto w = (r - l) / 2; w > 0; w /= 2) {
    while (p + w < n && ok(p + w)) {
        p += w;
    }
}
return ok(p)? p : -1; // 检查全false的情况
```

4. 最后一个不满足条件的元素: F F **F** T T

```c++
auto p = l;
for (auto w = (r - l) / 2; w > 0; w /= 2) {
    while (p + w < n && !ok(p + w)) {
        p += w;
    }
}
return !ok(p)? p : -1; // 检查全true的情况
```

注意到实际上1和2、3和4实际上只是取了个反, 也就是实际上只用两种就好.