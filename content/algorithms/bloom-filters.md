+++
title = "Bloom Filters"
weight = 1
order = 1
date = 2020-02-10
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]

+++


之前看Andy的数据库课程的时候提了一下，
大概就是一个压缩版的`bool`数组，
用来判断元素是否存在集合中。

<img src="https://raw.githubusercontent.com/vtta/assets/vtta.github.io/PicGo20200210175607.png" style="zoom:120%;filter:invert(100%);" />

- 数: $a$
- 哈希函数: $h_1(a), h_2(a), ..., h_k(a)$
- 比特向量: $m$位
- 插入$n$个数后某位为0的概率: $(1 - \frac{1}{m})^{kn}$
- 插入$n$个数后某位为1的概率: $1 - (1 - \frac{1}{m})^{kn}$
- 误报率: $P(k位都为1) = (1 - (1 - \frac{1}{m})^{kn})^k \approx (1 - e^{\frac{kn}{m}})^k$
- 右边极值点: $k=\frac{m}{n}ln2$
- 误报率最小值: $((\frac{1}{2})^{ln2})^\frac{m}{n}$
- 令$b = m/n$
  - 含义: 每个数在bloom filter中占的位数
- $P \approx (1-e^{\frac{k}{b}})^k$

