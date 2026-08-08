# 第56章 STL进阶、bitset与类基础
`[CSP-S 2025新增]` 难度5-6

!!! info "本章内容"
    本章讲解 STL 关联容器（map/set/multimap/multiset/unordered_map/unordered_set/deque）的原理与使用、常用算法函数，2025年 CSP-S 大纲新增的 bitset 位运算优化，以及面向对象编程中 class 的基本语法。这些内容是 CSP-S 2025 大纲的重要新增部分。

---

## 📌 学习目标

1. 能够说出 map、set、unordered_map、unordered_set 的底层实现（红黑树/哈希表）及各自时间复杂度
2. 能够正确使用 lower_bound、upper_bound、unique、next_permutation 等常用算法函数
3. 能够使用 bitset 进行位运算，掌握 count()、set()、reset()、flip() 等成员函数
4. 能够用 bitset 优化01背包，使时间复杂度除以 64（或 32）
5. 能够用 bitset 求图的可达性传递闭包
6. 能够定义 class，使用构造函数、public/private 访问控制，封装一个栈或队列

## 💡 生活类比

!!! tip "生活类比：字典、开关排与设计图纸"
    **map** 像"字典"——所有词条按字母顺序排列，查找某个词时可以快速定位（$O(\log n)$），而不需要从头翻到尾。

    **bitset** 像"一排开关"——一个 bitset 就是一排灯泡开关，每个开关只有开/关两种状态。你可以整体翻转、整体与运算，速度比逐个操作快 32 或 64 倍，因为 CPU 一条指令就能处理一整个字的位。

    **class** 像"设计图纸"——图纸定义了"汽车"该有哪些零件（成员变量）和功能（成员函数）。根据图纸可以造出很多辆汽车（对象），每辆车有独立的零件但共享同一套设计。

## 核心内容

### 56.1 STL关联容器总览

| 容器 | 底层实现 | 是否有序 | 是否去重 | 查找复杂度 |
|------|---------|---------|---------|-----------|
| `set` | 红黑树 | 有序（升序） | 去重 | $O(\log n)$ |
| `map` | 红黑树 | 按 key 有序 | key 去重 | $O(\log n)$ |
| `multiset` | 红黑树 | 有序 | 允许重复 | $O(\log n)$ |
| `multimap` | 红黑树 | 按 key 有序 | key 允许重复 | $O(\log n)$ |
| `unordered_set` | 哈希表 | 无序 | 去重 | $O(1)$ 平均 |
| `unordered_map` | 哈希表 | 无序 | key 去重 | $O(1)$ 平均 |

### 56.2 map与set的使用

**map** 是有序键值对，底层红黑树，支持按 key 遍历（升序）。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    map<string, int> mp;
    mp["apple"] = 3;
    mp["banana"] = 5;
    mp["apple"]++;              // 修改已有 key
    // 遍历（按 key 字典序升序）
    for (auto& p : mp)          // C++14 兼容写法
        cout << p.first << ":" << p.second << "\n";
    // 查找
    if (mp.count("banana")) cout << "found\n";
    // lower_bound: 第一个 >= key 的迭代器
    auto it = mp.lower_bound("ap");
    if (it != mp.end()) cout << it->first << "\n";
    return 0;
}
// 输入：（无输入）
// 输出：apple:4
//       banana:5
//       found
//       apple
```

**set** 是有序集合，自动去重排序。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    set<int> s = {3, 1, 4, 1, 5, 9, 2, 6};
    for (int x : s) cout << x << " ";
    cout << "\n";
    cout << "size=" << s.size() << "\n";       // 去重后 7 个
    s.erase(4);                                  // 删除元素 4
    auto it = s.find(5);
    if (it != s.end()) cout << "5 exists\n";
    return 0;
}
// 输入：（无输入）
// 输出：1 2 3 4 5 6 9
//       size=7
//       5 exists
```

### 56.3 multiset与multimap

`multiset` 允许重复元素，`multimap` 允许重复 key。注意 `multiset::erase(x)` 会删除**所有**等于 x 的元素，若只想删一个需用 `erase(iter)`。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    multiset<int> ms = {1, 1, 2, 2, 2, 3};
    cout << "count(2)=" << ms.count(2) << "\n";    // 3
    // 只删一个 2
    auto it = ms.find(2);
    if (it != ms.end()) ms.erase(it);
    cout << "after erase one: count(2)=" << ms.count(2) << "\n"; // 2
    return 0;
}
// 输入：（无输入）
// 输出：count(2)=3
//       after erase one: count(2)=2
```

### 56.4 unordered_map与unordered_set

哈希表实现，平均 $O(1)$，最坏 $O(n)$（哈希冲突被构造数据卡住时）。在 CSP 中若数据有被卡的风险，可使用 `map`/`set` 或自定义哈希。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    unordered_map<int, int> cnt;
    int a[] = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
    for (int x : a) cnt[x]++;
    for (auto& p : cnt) cout << p.first << "->" << p.second << "\n";
    return 0;
}
// 输入：（无输入）
// 输出：（顺序不定，但每行格式为 key->count，例如）
//       9->1
//       2->1
//       1->2
//       ...
```

### 56.5 deque双端队列

`deque` 支持头尾两端的 $O(1)$ 插入删除，也支持随机访问。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    deque<int> dq;
    dq.push_back(1); dq.push_back(2);
    dq.push_front(0);
    for (int x : dq) cout << x << " ";        // 0 1 2
    cout << "\n";
    dq.pop_back(); dq.pop_front();
    cout << dq.front() << "\n";               // 1
    return 0;
}
// 输入：（无输入）
// 输出：0 1 2
//       1
```

### 56.6 常用算法函数

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};
    // sort
    sort(v.begin(), v.end());                   // 1 1 2 3 4 5 6 9
    // lower_bound: 第一个 >= x 的位置
    auto lo = lower_bound(v.begin(), v.end(), 4);
    cout << "lower_bound(4)=" << (lo - v.begin()) << "\n";  // 4
    // upper_bound: 第一个 > x 的位置
    auto up = upper_bound(v.begin(), v.end(), 4);
    cout << "upper_bound(4)=" << (up - v.begin()) << "\n";  // 5
    // unique 去重（需先排序），返回去重后尾迭代器
    v = {1, 1, 2, 2, 3, 3};
    sort(v.begin(), v.end());
    auto last = unique(v.begin(), v.end());
    v.erase(last, v.end());                     // 1 2 3
    for (int x : v) cout << x << " ";
    cout << "\n";
    // next_permutation: 下一个排列
    vector<int> p = {1, 2, 3};
    do {
        for (int x : p) cout << x;
        cout << " ";
    } while (next_permutation(p.begin(), p.end()));
    cout << "\n";
    return 0;
}
// 输入：（无输入）
// 输出：lower_bound(4)=4
//       upper_bound(4)=5
//       1 2 3
//       123 132 213 231 312 321
```

### 56.7 bitset基础

`bitset<N>` 是一个固定大小的位集合，每个元素占 1 bit，整体操作利用 CPU 字长（32/64 位）并行处理，极为高效。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    bitset<8> b(13);                  // 13 = 00001101
    cout << b << "\n";                // 00001101
    cout << "count=" << b.count() << "\n";     // 1 的个数 = 3
    cout << "any=" << b.any() << "\n";         // 是否有 1 = 1
    cout << "none=" << b.none() << "\n";       // 是否全 0 = 0
    b.set(1);                         // 第 1 位设为 1 -> 00001111
    cout << b << "\n";
    b.reset(0);                       // 第 0 位设为 0 -> 00001110
    cout << b << "\n";
    b.flip(7);                        // 翻转第 7 位 -> 10001110
    cout << b << "\n";
    // 位运算
    bitset<8> x(12), y(10);           // 00001100, 00001010
    cout << (x & y) << "\n";          // 00001000
    cout << (x | y) << "\n";          // 00001110
    cout << (x ^ y) << "\n";          // 00000110
    cout << (x << 2) << "\n";         // 00110000
    cout << (x >> 2) << "\n";         // 00000011
    return 0;
}
// 输入：（无输入）
// 输出：00001101
//       count=3
//       any=1
//       none=0
//       00001111
//       00001110
//       10001110
//       00001000
//       00001110
//       00000110
//       00110000
//       00000011
```

### 56.8 bitset优化01背包

01背包转移：$f[j] = f[j] \lor f[j-w_i]$（可行性背包，只问能否凑出）。
用 bitset 表示 $f$，转移变成 `f |= (f << w_i)`，一条指令处理 64 位，**时间优化约 64 倍**。

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int n, W;
    cin >> n >> W;
    bitset<100001> f;
    f[0] = 1;                         // 0 重量一定能凑出
    for (int i = 1; i <= n; i++) {
        int w;
        cin >> w;
        f |= (f << w);                // 核心转移
    }
    int ans = 0;
    for (int i = 0; i <= W; i++) if (f[i]) ans++;  // 只统计 ≤W 的可达重量
    cout << ans << "\n";              // 能凑出的重量种数
    return 0;
}
// 输入：4 10
//       2 3 5 7
// 输出：8
// 说明：在 W=10 范围内能凑出 0,2,3,5,7,8,9,10 共 8 种
```

### 56.9 bitset求可达性传递闭包

求有向图中所有点对之间的可达性（传递闭包）：对每个点 $i$ 维护一个 bitset $R_i$，$R_i[j]=1$ 表示 $i$ 能到 $j$。按拓扑序或直接多次松弛：$R_i = \bigvee_{i \to j} R_j \cup \{i\}$。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 505;
bitset<N> reach[N];
vector<int> g[N];
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
    }
    // 倒序 DP（适用于 DAG，按拓扑逆序处理）
    // 这里用简单 Floyd-bitset 适用于一般情况
    for (int i = 1; i <= n; i++) reach[i][i] = 1;
    for (int k = 1; k <= n; k++)
        for (int i = 1; i <= n; i++)
            if (reach[i][k]) reach[i] |= reach[k];
    // reach[i][j]==1 表示 i 能到 j
    int q;
    cin >> q;
    while (q--) {
        int u, v;
        cin >> u >> v;
        cout << (reach[u][v] ? "Yes" : "No") << "\n";
    }
    return 0;
}
// 输入：4 4
//       1 2
//       2 3
//       3 4
//       1 3
//       3
//       1 4
//       4 1
//       2 4
// 输出：Yes
//       No
//       Yes
```

### 56.10 类与对象基础

#### struct 到 class 的过渡

C++ 中 `struct` 和 `class` 几乎等价，唯一区别是默认访问权限：`struct` 默认 `public`，`class` 默认 `private`。

```cpp
#include <bits/stdc++.h>
using namespace std;

// 用 class 封装一个栈
class Stack {
private:
    int data[100005];
    int topIdx;
public:
    // 构造函数：初始化
    Stack() : topIdx(0) {}
    void push(int x) { data[++topIdx] = x; }
    void pop() { topIdx--; }
    int top() const { return data[topIdx]; }
    bool empty() const { return topIdx == 0; }
    int size() const { return topIdx; }
};

int main() {
    Stack s;
    s.push(10);
    s.push(20);
    s.push(30);
    cout << "size=" << s.size() << "\n";    // 3
    cout << "top=" << s.top() << "\n";      // 30
    s.pop();
    cout << "top=" << s.top() << "\n";      // 20
    return 0;
}
// 输入：（无输入）
// 输出：size=3
//       top=30
//       top=20
```

#### 成员变量、成员函数、构造函数、访问控制

```cpp
#include <bits/stdc++.h>
using namespace std;

class Queue {
private:                        // 私有：外部不能直接访问
    int data[100005];
    int head, tail;
public:                         // 公有：外部可调用
    Queue() {                   // 构造函数：对象创建时自动调用
        head = 1;
        tail = 0;
    }
    void push(int x) {
        data[++tail] = x;
    }
    void pop() {
        head++;
    }
    int front() const {         // const 表示不修改成员
        return data[head];
    }
    bool empty() const {
        return head > tail;
    }
    int size() const {
        return tail - head + 1;
    }
};

int main() {
    Queue q;
    q.push(1);
    q.push(2);
    q.push(3);
    while (!q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    cout << "\n";
    return 0;
}
// 输入：（无输入）
// 输出：1 2 3
```

## 常见误区

1. **误区**：以为 `unordered_map` 一定比 `map` 快。
   **纠正**：平均 $O(1)$ 但常数较大，且最坏 $O(n)$ 可被构造数据卡。小数据量或需要有序遍历时用 `map` 更稳妥。

2. **误区**：`multiset::erase(x)` 只删一个元素。
   **纠正**：传值会删除所有等于 x 的元素；要删一个需传迭代器 `erase(find(x))`。

3. **误区**：`unique` 能直接去重。
   **纠正**：`unique` 只去掉**相邻**重复元素，必须先 `sort`，且要用返回值 `erase` 掉尾部。

4. **误区**：bitset 优化背包时忘记初始化 `f[0]=1`。
   **纠正**：`f[0]=1` 表示"重量 0 一定能凑出"，是递推的起点。

5. **误区**：在 class 中把成员变量设为 `public` 并在外部直接修改。
   **纠正**：封装的意义在于隐藏实现细节，应通过成员函数访问，保持数据一致性。

6. **误区**：`lower_bound` 用于 `unordered_set`。
   **纠正**：`lower_bound` 要求容器有序，`unordered_*` 无序，不能使用。应改用 `set` 或对 `vector` 排序后使用。

## 典型例题

### 例题1：P3674 小清新人渣的本愿（bitset优化）

**题意**：给定一个长为 $n$ 的数列，$m$ 次询问，每次询问区间 $[l,r]$ 内是否能选出两个数使得它们的和/差/积等于 $x$。

**思路**：用 bitset 维护数列中出现的数（值域 $[0, V]$）。对于每个询问的区间，用莫队维护当前区间内出现的数集合 `now`（bitset）。差为 $x$ 即存在 $a, a-x$ 同时出现，即 `(now & (now >> x)).any()`；和为 $x$ 即存在 $a, x-a$ 同时出现，用反序 bitset 辅助判断；积为 $x$ 则枚举约数。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100005, V = 100005;
int n, m, blk;
int a[N], cnt[V], ans[N];
bitset<V> now, rev;      // now[i]=1 表示值 i 在当前区间出现；rev 是反序辅助
struct Q { int l, r, op, x, id; } q[N];
bool cmp(const Q&a, const Q&b) {
    if (a.l/blk != b.l/blk) return a.l < b.l;
    return (a.l/blk & 1) ? a.r < b.r : a.r > b.r;
}
inline void add(int v) { if (++cnt[v] == 1) now[v] = 1, rev[V-1-v] = 1; }
inline void del(int v) { if (--cnt[v] == 0) now[v] = 0, rev[V-1-v] = 0; }
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m;
    blk = max(1, (int)(n / sqrt(m)));
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 1; i <= m; i++) {
        cin >> q[i].op >> q[i].l >> q[i].r >> q[i].x;
        q[i].id = i;
    }
    sort(q+1, q+m+1, cmp);
    int l = 1, r = 0;
    for (int i = 1; i <= m; i++) {
        while (r < q[i].r) add(a[++r]);
        while (l > q[i].l) add(a[--l]);
        while (r > q[i].r) del(a[r--]);
        while (l < q[i].l) del(a[l++]);
        int op = q[i].op, x = q[i].x;
        if (op == 1) { // 差为 x：存在 a, a-x
            if ((now & (now >> x)).any()) ans[q[i].id] = 1;
        } else if (op == 2) { // 和为 x：存在 a, x-a，用 rev
            // a + b = x  =>  a = x - b；now 中 a 与 rev[V-1-b] 对应
            // rev[v]=1 当且仅当值 (V-1-v) 出现，即 now[V-1-v]=1
            // 若 b 出现，则 rev[V-1-b]=1；要 a=x-b 出现即 now[x-b]=1
            // 即 now[x-b] 与 now[b] 同时为1，等价于检查 now 和 (rev>>(V-1-x)).any()
            // 简化：构造 tmp 使得 tmp[i]=now[x-i]，用 rev 实现
            bitset<V> tmp;
            // now[x-i] 用 rev: rev[j]=1 当 now[V-1-j]=1
            // 令 V-1-j = x-i => j = V-1-x+i => tmp = rev >> (V-1-x)
            if (x >= V) continue;  // 防止位移为负（未定义行为）
            tmp = rev >> (V - 1 - x);
            if ((now & tmp).any()) ans[q[i].id] = 1;
        } else { // 积为 x：枚举约数
            for (int d = 1; (long long)d * d <= x; d++) {
                if (x % d == 0) {
                    int e = x / d;
                    if (now[d] && now[e]) { ans[q[i].id] = 1; break; }
                }
            }
        }
    }
    for (int i = 1; i <= m; i++) cout << (ans[i] ? "hana" : "bi") << "\n";
    return 0;
}
// 输入：5 3
//       1 2 3 4 5
//       1 1 3 1
//       2 1 4 5
//       3 2 5 6
// 输出：hana
//       hana
//       hana
```

### 例题2：P1531 I Hate It（ST表求区间最大值）

**题意**：给定 $n$ 个学生成绩，$m$ 次询问区间 $[l,r]$ 内的最大成绩。

**思路**：用 ST 表预处理 $O(n \log n)$，每次查询 $O(1)$。也可用线段树。这里用 ST 表实现，简洁高效。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 200005, LOG = 20;
int a[N], st[N][20], lg[N];
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> a[i];
    // 预处理 log 表
    lg[1] = 0;
    for (int i = 2; i <= n; i++) lg[i] = lg[i/2] + 1;
    // 初始化 ST 表
    for (int i = 1; i <= n; i++) st[i][0] = a[i];
    for (int j = 1; (1 << j) <= n; j++)
        for (int i = 1; i + (1 << j) - 1 <= n; i++)
            st[i][j] = max(st[i][j-1], st[i + (1 << (j-1))][j-1]);
    // 查询
    while (m--) {
        int l, r;
        cin >> l >> r;
        int k = lg[r - l + 1];
        cout << max(st[l][k], st[r - (1 << k) + 1][k]) << "\n";
    }
    return 0;
}
// 输入：5 3
//       1 5 3 4 2
//       1 3
//       2 5
//       4 4
// 输出：5
//       5
//       4
```

## 📖 本章总结

- STL进阶、bitset与类基础
  - STL关联容器
    - map/set：红黑树，有序，$O(\log n)$
    - multiset/multimap：允许重复
    - unordered_map/unordered_set：哈希，$O(1)$ 平均
    - deque：双端队列
  - 常用算法
    - sort, lower_bound, upper_bound
    - unique（需先排序）, next_permutation
  - bitset（2025新增）
    - 定义与位运算 & | ^ ~ << >>
    - 成员函数 count/any/none/set/reset/flip
    - 优化01背包：`f |= (f << w)`，提速 64 倍
    - 传递闭包：`reach[i] |= reach[k]`
  - 类与对象基础
    - struct → class（默认 private）
    - 成员变量与成员函数
    - 构造函数初始化
    - public/private 访问控制
    - 封装栈/队列示例

### 知识衔接

本章是考纲补强卷的第六章，与前后章节的联系如下：

- **前置知识**：第06章数组（STL容器的底层存储）、第09章结构体与文件操作（class是结构体的进阶）、第16章位运算与离散化（bitset的位运算基础）
- **本章定位**：讲解STL关联容器进阶、bitset位运算优化与面向对象class入门，是2025年CSP-S新增考纲的重要内容
- **后续发展**：本章为第57章概率与期望DP铺路，STL容器与class封装将在后续DP代码的组织中发挥作用

!!! info "下一章预告"
    下一章我们将学习 **概率与期望DP**，包括概率公理、条件概率、期望的线性性，以及从终点倒推的期望DP和从起点正推的概率DP。这是 CSP-S 进阶内容，涉及掷骰子、走迷宫等经典模型。
