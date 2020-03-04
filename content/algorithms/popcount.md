+++
title = "Popcount"
weight = 1
order = 1
date = 2020-03-04
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]
+++

`popcount(x)`就是计算一个数`x`的二进制表示里面有几个1。

GCC里面有个内置函数叫`__builtin_popcount(x)`就是实现的这个功能。

一般想到的办法就是一位一位和1与，计算1的个数。

有一个比较巧妙的方法：
假设`x`的二进制表示是`A1B`
其中`B`全为`0`，则`x-1`的二进制表示是`A0(~B)`
那么`x & (x-1) = (A1B) & (A0(~B)) = A0(~B)`
其中`~B`全为1，也就是构造了一个mask，用中间的0抹去了中间的1，这个1也是`x`中最后一个`1`。
循环`x &= (x-1)`会每次都抹去`x`中的一个1，
重复下去直到`x`为`0`所经过的次数也就是`x`中`1`的个数。

```c++
auto popcount(unsigned x) {
    auto count = 0U;
    while (x != 0U) {
        x &= (x - 1);
        ++count;
    }
    return count;
}
```