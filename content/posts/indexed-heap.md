+++
title = "Indexed Heap"
weight = 1
order = 1
date = 2020-02-06
#insert_anchor_links = "right"

[taxonomies]
tags = ["algorithm"]
+++

昨天写[POJ2485](@/posts/poj-2485.md)的时候
看那一个循环里面又套一个循环的，心里难受。
一直说Prim可以用堆优化，
于是来试试。

最先写但时候发现普通的堆还不行，
因为每次从堆中取出最小`key`值的节点时得知道它是哪个节点，
也就是得知道下标。
于是想为啥不能用`map+heap`来整呢，
一查，还真有这么个东西，
就叫索引堆。

索引堆的思想很简单，
为了想知道根据存的值`value`找到下标`key`，
那么就另外维护一个数据结构记录这个对应关系。

实现时，
索引堆中的数据其实全部存放在一个数组中，
这样就能寻下标`key`访问。
即将原来只有值的元素看成是`key-value pair`。
因为反正值也不会单独存放，
肯定一般是在一个数组里面，
那何必不把这个数组的下标当成`key`呢，
这个数组就叫`data`好了。

那么说好的堆呢？
这时用另外一个数组`position`来记录下标为`key`的节点到底在堆中的哪一个位置。

好，堆的问题解决了，
但是最开始的问题，假如知道了堆中的某个节点，
或者说知道了堆中位置`index`，
怎么找到对应的`key`呢？
简单，再来一个数组`inverse`，
反向将堆中位置映射到数据下标。

总的来说，普通堆是一个数组，
索引堆是三个数组`data`，`position`，`inverse`：
- `data`存放元素值
- `position`存放元素在堆中的位置
- `inverse`将堆中的位置映射回元素的`key`值

几个简单的场景：
- 根据元素下标取值：`data[key]`
- 根据元素下标找到堆中位置：`position[key]`
- 根据堆中位置取下标：`inverse[index]`
- 根据堆中位置取值：`data[inverse[index]]`
- 从堆中的位置绕一圈再回来：`position[inverse[index]] == index`
- 从下标开始绕一圈再回来：`inverse[position[key]] == key`

[偶然发现《算法》里面取名字还听有意思的哦](https://algs4.cs.princeton.edu/24pq/)，
忍不住复习一下堆的操作：
- 上移，不不不，明明叫上浮，哦不，游泳
  - 每次和节点的父亲比较，更小则交换
  - 注意：用`i`和`j`表示他们的含义是堆中位置，`k`则用来表示`key`
  - 注意：因为我们`data`数组维持了`key`和值的对应关系，交换的其实是他们在堆中的位置
  - 比较其实也是先根据堆中位置找到元素值后再比较
  ```c++
  void swim(size_type i) {
      size_type j;
      while (i > 0 && less(i, j = parent(i))) {
          exch(i, j);
          i = j;
      }
  }
  ```
- 下移，哦豁，沉了
  - 先将左右孩子比较，保留最小孩子的堆中位置
  - 保证了总是和最小的孩子交换
  ```c++
  void sink(size_type i) {
      size_type j;
      while ((j = left(i)) < size()) {
          if (j + 1 < size() && less(j + 1, j)) {
              j += 1;
          }
          if (less(i, j)) {
              break;
          }
          exch(i, j);
          i = j;
      }
  }
  ```
- 弗洛伊德建堆，$O(n)$
  - 从最后一个非叶子节点开始
  - 即每次下移操作实际上等价于合并左右子堆
  ```c++
  void heapify() {
      for (size_type i = size() / 2; i > 0; --i) {
          sink(i - 1);
      }
  }
  ```
- 插入
  - （因为想做成可以扩展的数据结构，写得很丑）
  - `data`数组的大小永远是曾经插入过元素的最大`key`值+1
  - 刚放进`data`数组的元素在堆中应该被放在最后
  - 更新反向映射数组后将元素在堆中上移
  ```c++
  void insert(size_type k, const_reference v) {
      if (k >= data.size()) {
          data.resize(k + 1);
      }
      size_type x;
      while (k >= (x = position.size())) {
          position.resize(2 * x + 1, inf);
      }
      while (sz >= (x = inverse.size())) {
          inverse.resize(2 * x + 1, inf);
      }
      data[k] = v;
      position[k] = sz;
      inverse[sz] = k;
      swim(sz);
      sz += 1;
  }
  ```
- 删除
  - 将元素与堆中末尾元素互换
  - 换过去的元素要上移还是下移不知道，就都试一下
  - 删除指的是只从堆中删除
  - 即元素数组中这个元素还在
  ```c++
  void remove(size_type k) {
      size_type i = position[k];
      sz -= 1;
      exch(i, sz);
      swim(i);
      sink(i);
      // data[k] is not actually deleted
      position[k] = inf;
      inverse[sz] = inf;
  }
  ```

用了索引堆的Prim算法：
```c++
void prim() {
    heap.update(0, 0);
    for (int i = 1; i < n; ++i) {
        int u = heap.peek_key();
        heap.pop();
        mst[u] = 1;
        for (int v = 0; v < n; ++v) {
            if (mst[v] == 0 && g[u][v] != 0 
                && g[u][v] < heap[v]) {
                parent[v] = u;
                heap.update(v, min(g[u][v], heap[v]));
            }
        }
    }
}
```

索引堆的完整代码：
```c++
// key: from outside the heap, if users see the heap as a map
// index: the index in the logical heap, it's inside the implementation
// min heap in c++03 for poj
template <typename T, typename C = std::vector<T>, typename Cmp = std::less<T> >
class IndexedHeap {
public:
    typedef T value_type;
    typedef C container_type;
    typedef Cmp value_compare;
    typedef std::size_t size_type;
    typedef T &reference;
    typedef T const &const_reference;
    typedef T *pointer;
    typedef T const *const_pointer;

    IndexedHeap(container_type const &c = container_type())
        : inf(-1),
          sz(c.size()),
          cmp(),
          data(c),
          position(sz, inf),
          inverse(sz, inf) {
        for (size_type i = 0; i < size(); ++i) {
            position[i] = i;
            inverse[i] = i;
        }
        heapify();
    }
    
    const_reference at(size_type k) { return data.at(k); }
    const_reference operator[](size_type k) { return data[k]; }
    size_type size() const { return sz; }
    bool empty() const { size() == 0; }

    bool contains(size_type k) const {
        return k < size() && position[k] != inf;
    }

    const_reference top() const { return data[inverse[0]]; }

    size_type top_key() const { return inverse[0]; }
    
    void pop() { remove(top_key()); }
    
    size_type push(const_reference v) {
        size_type k = data.size();
        insert(k, v);
        return k;
    }
    
    void insert(size_type k, const_reference v) {
        if (k >= data.size()) {
            data.resize(k + 1);
        }
        size_type x;
        while (k >= (x = position.size())) {
            position.resize(2 * x + 1, inf);
        }
        while (sz >= (x = inverse.size())) {
            inverse.resize(2 * x + 1, inf);
        }
        data[k] = v;
        position[k] = sz;
        inverse[sz] = k;
        swim(sz);
        sz += 1;
    }
    
    void update(size_type k, const_reference v) {
        size_type i = position[k];
        bool increase = cmp(data[k], v);
        data[k] = v;
        if (increase) {
            sink(i);
        } else {
            swim(i);
        }
    }
    
    void remove(size_type k) {
        size_type i = position[k];
        sz -= 1;
        exch(i, sz);
        swim(i);
        sink(i);
        // data[k] is not actually deleted
        position[k] = inf;
        inverse[sz] = inf;
    }

private:
    const size_type inf;
    size_type sz;
    value_compare cmp;
    container_type data;
    // position map (key -> index)
    // which maps a given key to the node position in the heap
    std::vector<size_type> position;
    // inverse map (index -> key)
    // which maps the node position in the heap to the key
    std::vector<size_type> inverse;

    static size_type parent(size_type i) { return (i - 1) / 2; }
    static size_type left(size_type i) { return i * 2 + 1; }
    static size_type right(size_type i) { return i * 2 + 2; }

    bool less(size_type i, size_type j) {
        return cmp(data[inverse[i]], data[inverse[j]]);
    }

    void exch(size_type i, size_type j) {
        position[inverse[i]] = j;
        position[inverse[j]] = i;
        std::swap(inverse[i], inverse[j]);
    }

    void swim(size_type i) {
        size_type j;
        while (i > 0 && less(i, j = parent(i))) {
            exch(i, j);
            i = j;
        }
    }

    void sink(size_type i) {
        size_type j;
        while ((j = left(i)) < size()) {
            if (j + 1 < size() && less(j + 1, j)) {
                j += 1;
            }
            if (less(i, j)) {
                break;
            }
            exch(i, j);
            i = j;
        }
    }

    void heapify() {
        for (size_type i = size() / 2; i > 0; --i) {
            sink(i - 1);
        }
    }
};
```

POJ2485余下代码:
```c++
// https://vjudge.net/problem/POJ-2485
#include <iostream>
#include <limits>
#include <vector>
using namespace std;

const int INF = numeric_limits<int>::max();

struct Solution {
    int n;
    vector<vector<int> > g;
    vector<int> mst, parent;
    IndexedHeap<int> heap;

    Solution(int n)
        : n(n), g(n), mst(n, 0), parent(n, -1), heap(vector<int>(n, INF)) {
        for (int i = 0; i < n; ++i) {
            g[i].resize(n);
            for (int j = 0; j < n; ++j) {
                cin >> g[i][j];
            }
        }
    }
    void solve() {
        prim();

        int longest = -INF;
        for (int i = 1; i < n; ++i) {
            longest = max(longest, g[parent[i]][i]);
        }
        cout << longest << endl;
    }
    void prim() {
        heap.update(0, 0);
        // n - 1 vertices left to expand to
        for (int i = 1; i < n; ++i) {
            int u = heap.peek_key();
            heap.pop();
            mst[u] = 1;
            for (int v = 0; v < n; ++v) {
                if (mst[v] == 0 && g[u][v] != 0 && g[u][v] < heap[v]) {
                    parent[v] = u;
                    heap.update(v, min(g[u][v], heap[v]));
                }
            }
        }
    }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int T;
    cin >> T;
    for (int i = 0; i < T; ++i) {
        int n;
        cin >> n;
        Solution(n).solve();
    }
}
```