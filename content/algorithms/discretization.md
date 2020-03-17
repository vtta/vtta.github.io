+++
title = "Discretization"
weight = 1
order = 1
date = 2020-03-17
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithms"]
+++

在处理数组时，
如果数组中的数字只有相对大小有意义，
但是数据太过分散，
不好直接用于数组下标时，
可以使用离散化，
将数值映射到$[0, n)$内，
方便处理。
配合树状数组食用更佳。

题目：
- [数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

离散化的复杂度是$O(nlogn)$，复杂度要求太高的题目要注意了。
思路很简单：
- 将原数组复制一遍后排序取去重
- 新数组中数值和下标就是我们要的映射关系

代码：
```c++
void discretization(vector<int>& nums) {
    vector<int> t(nums);
    sort(begin(t), end(t));
    auto pos = unique(begin(t), end(t));
    for (auto & i : nums) {
        i = lower_bound(begin(t), pos, i) - begin(t);
    }
}
```

离散化 + Fenwick tree求逆序对：
```c++
class Solution {
    vector<int> tree;
public:
    int reversePairs(vector<int>& nums) {
        auto n = nums.size();
        if (n <= 1) {
            return 0;
        }
        discretization(nums);
        // 树状数组的第一个元素不能用，整体后移一个位置
        tree.resize(n + 1);
        auto inv = 0;
        for (auto const & i : nums) {
            update(i, 1);
            inv += sum(n) - sum(i);
        }
        return inv;
    }
    
    void discretization(vector<int>& nums) {
        vector<int> t(nums);
        sort(begin(t), end(t));
        auto pos = unique(begin(t), end(t));
        for (auto & i : nums) {
            // 树状数组的第一个元素不能用，整体后移一个位置
            i = lower_bound(begin(t), pos, i) - begin(t) + 1;
        }
    }
    
    void update(int x, int delta) {
        auto n = tree.size();
        while (x < n) {
            tree[x] += delta;
            x += lowbit(x);
        }
    }
    
    int sum(int x) {
        int ret = 0;
        while (x > 0) {
            ret += tree[x];
            x -= lowbit(x);
        }
        return ret;
    }
    
    int lowbit(int x) {
        return x & -x;
    }
};
```

另外附上归并排序的解法：
```c++
class Solution {
public:
    int reversePairs(vector<int>& nums) {
        auto n = nums.size();
        if (n <= 1) {
            return 0;
        }
        auto t = vector<int>(n);
        return sort(nums, 0, n, t);
    }
    
    int sort(vector<int>& nums, int l, int r, vector<int>& t) {
        if (r - l <= 1) {
            return 0;
        }
        auto m = l + (r - l) / 2;
        auto inv = 0;
        inv += sort(nums, l, m, t);
        inv += sort(nums, m, r, t);
        auto p = l, q = m, k = l;
        // 1 2 5 7 | 0 3 6 9 
        //     ^       ^
        while (p < m && q < r) {
            if (nums[p] <= nums[q]) {
                t[k] = nums[p];
                k += 1, p += 1;
            } else {
                inv += m - p;
                t[k] = nums[q];
                k += 1, q += 1;
            }
        }
        while (p < m) {
            t[k] = nums[p];
            k += 1, p += 1;
        }
        while (q < r) {
            t[k] = nums[q];
            k += 1, q += 1;
        }
        copy(cbegin(t) + l, cbegin(t) + r, begin(nums) + l);        
        return inv;
    } 
};
```
