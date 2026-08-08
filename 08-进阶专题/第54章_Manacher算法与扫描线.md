# 第54章 Manacher算法与扫描线

`[CSP-S 2025新增]` 难度7

!!! info "本章内容"
    - 回文串的基本概念与朴素算法的缺陷
    - Manacher 算法核心思想：利用回文对称性加速
    - 插入分隔符 `#` 统一奇偶长度
    - P 数组（回文半径）的定义与计算
    - Manacher 线性时间复杂度证明
    - 扫描线算法基本思想：用一条线扫过平面维护状态
    - 矩形面积并问题与坐标离散化
    - 线段树配合扫描线求面积并
    - 扫描线求矩形周长并（简介）

---

## 📌 学习目标

学完本章后，你应当能够：

1. **准确说出**回文串的定义，并解释朴素算法为何是 $O(n^2)$
2. **默写** Manacher 算法的完整实现，包括插入分隔符、`P` 数组计算、`mr`（右边界）维护
3. **解释**为何插入 `#` 能统一奇偶长度回文，并推导 Manacher 的 $O(n)$ 复杂度
4. **运用** Manacher 算法在 $O(n)$ 时间内求出最长回文子串
5. **描述**扫描线的基本思想，并能对矩形面积并问题进行坐标离散化与建线段树
6. **默写**扫描线 + 线段树求矩形面积并的完整代码，理解"lazy 标记"与"长度统计"的配合

---

## 💡 生活类比

!!! tip "生活类比：照镜子与安检扫描仪"
    **Manacher 像照镜子**——你站在镜子前，如果左半边是对称的，右半边也大概率对称。Manacher 算法正是利用"已知回文的对称性"来推断新位置的回文半径，避免重复计算。

    **扫描线像安检扫描仪**——一条扫描线从左扫到右，经过的每个位置记录当前状态（如当前被覆盖的 y 轴总长度）。把矩形拆成"进入"和"离开"两个事件，扫描线依次处理这些事件，累加面积。

---

## 第一节 回文串与朴素算法

### 1.1 回文串的定义

「**回文串 Palindrome**」：正读与反读相同的字符串。如 `aba`、`abba`、`a` 都是回文串。

- **奇长度回文**：以某个字符为中心，如 `aba` 以 `b` 为中心。
- **偶长度回文**：以两个字符之间的空隙为中心，如 `abba` 以两个 `b` 之间为中心。

### 1.2 朴素算法

对每个中心位置 $i$，向两侧扩展，直到不匹配。时间复杂度 $O(n^2)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

string s;
int expand(int l, int r) {
    while (l >= 0 && r < (int)s.size() && s[l] == s[r]) {
        l--; r++;
    }
    return r - l - 1;   // 回文长度
}

int main() {
    cin >> s;
    int ans = 0;
    int n = s.size();
    for (int i = 0; i < n; i++) {
        ans = max(ans, expand(i, i));       // 奇长度
        ans = max(ans, expand(i, i + 1));  // 偶长度
    }
    cout << ans << "\n";
    return 0;
}
// 输入：ababa
// 输出：5
```

### 1.3 朴素算法的缺陷

每个中心最坏扩展 $O(n)$ 次（如全相同字符 `aaaa`），总共 $n$ 个中心，总复杂度 $O(n^2)$。当 $n = 10^6$ 时无法承受。

!!! warning "朴素算法的浪费"
    朴素算法没有利用"已知回文的对称性"。如果一段回文 `[l, r]` 已经确认，那么关于中心对称的另一段很可能也是回文——这就是 Manacher 加速的契机。

---

## 第二节 Manacher 算法

### 2.1 核心思想

Manacher 算法通过维护**已知的最长回文右边界**，利用回文的对称性，直接初始化新位置的回文半径，再向两侧扩展。最坏情况下每个字符的扩展均摊 $O(1)$，总复杂度 $O(n)$。

### 2.2 插入分隔符统一奇偶

为避免奇偶长度分别处理，在字符之间和首尾插入一个特殊字符 `#`：

```
原串：    a   b   a
新串：  # a # b # a #
原串：    a   b   b   a
新串：  # a # b # b # a #
```

插入后：
- 新串长度变为 $2n + 1$（恒为奇数）。
- 每个回文中心都对应一个字符位置，奇偶长度统一处理。
- 原串长度为 $k$ 的回文 → 新串中半径为 $k$（中心是字符）或 $k+1$（中心是 `#`）。

### 2.3 P 数组的定义

设新串为 $t$（下标从 0 开始），定义：

$$P[i] = \text{以 } t[i] \text{ 为中心的最长回文半径（含中心）}$$

即 $t[i-P[i]+1 \dots i+P[i]-1]$ 是回文，长度为 $2P[i]-1$。

**原串中的最长回文长度** $= \max(P[i]) - 1$。

```
新串 t:  #  a  #  b  #  a  #
下标 i:  0  1  2  3  4  5  6
P[i]:    1  2  1  4  1  2  1

最大 P[i] = 4（在 i=3 即 'b' 处）
原串最长回文长度 = 4 - 1 = 3（即 "aba"）
```

### 2.4 算法核心：利用对称性加速

维护两个变量：

- `mr`（max right）：当前已知回文能到达的最右边界。
- `mid`：`mr` 对应的回文中心。

当处理位置 $i$（$i < mr$）时，$i$ 关于 `mid` 的对称点为 $j = 2 \cdot mid - i$。根据对称性：

$$P[i] \ge \min(P[j], mr - i)$$

- 若 $P[j] < mr - i$：$P[i] = P[j]$，无需扩展。
- 若 $P[j] \ge mr - i$：$P[i] = mr - i$，从 `mr - i` 开始尝试扩展。
- 若 $i \ge mr$：无对称信息，$P[i] = 1$，从头扩展。

扩展成功后更新 `mr` 和 `mid`。

### 2.5 完整代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 11000005;
char t[MAXN * 2];
int P[MAXN * 2];

int manacher(const string &s) {
    int n = s.size();
    // 构造新串 t = #s[0]#s[1]#...#s[n-1]#
    int len = 0;
    t[len++] = '#';
    for (int i = 0; i < n; i++) {
        t[len++] = s[i];
        t[len++] = '#';
    }
    t[len] = '\0';
    // Manacher 主算法
    int mr = 0, mid = 0, ans = 0;
    for (int i = 0; i < len; i++) {
        if (i < mr) {
            P[i] = min(P[2 * mid - i], mr - i);   // 利用对称性
        } else {
            P[i] = 1;                              // 无对称信息
        }
        // 尝试扩展
        while (i - P[i] >= 0 && i + P[i] < len && t[i - P[i]] == t[i + P[i]]) {
            P[i]++;
        }
        // 更新右边界
        if (i + P[i] - 1 > mr) {
            mr = i + P[i] - 1;
            mid = i;
        }
        ans = max(ans, P[i] - 1);   // 原串回文长度 = P[i] - 1
    }
    return ans;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    string s;
    cin >> s;
    cout << manacher(s) << "\n";
    return 0;
}
// 输入：ababa
// 输出：5
// 输入：aaaa
// 输出：4
// 输入：abc
// 输出：1
```

### 2.6 线性时间复杂度证明

**关键观察**：`mr` 单调不减。

- 每次 `while` 扩展成功，意味着 `i + P[i] - 1` 超过了 `mr`，于是 `mr` 被更新为更大的值。
- `mr` 从 0 增长到最多 `len`，共增长 $O(n)$ 次。
- 因此所有扩展操作的总次数不超过 $O(n)$。
- 加上每个位置 $O(1)$ 的初始化，总复杂度 $O(n)$。

!!! info "Manacher 的精髓"
    朴素算法每个中心"独立扩展"，Manacher 通过 `mr` 这个全局指针，保证"扩展"只在 `mr` 之外发生，而 `mr` 只增不减，从而均摊 $O(1)$ 每个位置。这与 KMP、Z 函数的思想一脉相承。

### 2.7 复杂度

| 维度 | 复杂度 | 说明 |
|------|:------:|------|
| 时间 | $O(n)$ | `mr` 单调不减 |
| 空间 | $O(n)$ | 新串与 `P` 数组 |

---

## 第三节 扫描线算法

### 3.1 基本思想

「**扫描线算法 Scan Line**」：想象一条垂直（或水平）的线从左到右扫过平面，维护"当前扫描线穿过区域的某种状态"（如被覆盖的总长度），把问题分解为扫描线在每个"事件点"处更新状态并累计答案。

### 3.2 矩形面积并问题

**问题**：平面上有 $n$ 个矩形（边与坐标轴平行），求它们的**并集面积**。

**核心思路**：

1. 每个矩形 $[x_1, x_2] \times [y_1, y_2]$ 拆成两个事件：
   - 在 $x = x_1$ 处，y 区间 $[y_1, y_2]$ 的"覆盖次数 +1"。
   - 在 $x = x_2$ 处，y 区间 $[y_1, y_2]$ 的"覆盖次数 -1"。
2. 把所有事件按 $x$ 排序，扫描线从左到右依次处理。
3. 两个相邻事件 $x_i$ 与 $x_{i+1}$ 之间，扫描线"冻结"在某个 y 覆盖状态，贡献面积 = `当前被覆盖的 y 总长度 × (x_{i+1} - x_i)`。
4. 用线段树维护 y 轴上每个区间的"被覆盖长度"与"覆盖次数"。

### 3.3 坐标离散化

y 坐标值可能很大（如 $10^9$），需要离散化：

```
原始 y 坐标: y1, y2, y3, ...
排序去重后:   y[1], y[2], ..., y[k]
离散化后每个区间 [y[i], y[i+1]] 作为一个"基本单元"
```

离散化后，线段树维护 $k-1$ 个"基本区间"的覆盖状态。

### 3.4 线段树配合扫描线

线段树的每个节点维护：

- `cover`：该区间被完整覆盖的次数（**不下传**，因为加入和移除成对）。
- `len`：该区间内被覆盖的 y 总长度。

更新规则：

- 若 `cover > 0`：整个区间被覆盖，`len = y[r+1] - y[l]`（离散化后原始长度）。
- 若 `cover == 0` 且是叶子：`len = 0`。
- 若 `cover == 0` 且非叶子：`len = 左子.len + 右子.len`。

!!! tip "为什么 cover 不需要下传 lazy 标记"
    扫描线中，每个矩形的"加入"和"移除"是成对出现的，且按 $x$ 顺序处理。`cover` 记录的是"完整覆盖次数"，加 1 和减 1 是对称的，无需下传。这正是扫描线线段树的精妙之处。

### 3.5 完整代码（矩形面积并）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 200005;
int n;
double ys[MAXN * 2];     // 离散化用 y 数组
int yCnt = 0;

struct Event {
    double x, y1, y2;
    int kind;            // +1 加入, -1 移除
    bool operator<(const Event &o) const {
        return x < o.x;
    }
} ev[MAXN * 2];

struct Node {
    int cover;           // 完整覆盖次数
    double len;          // 被覆盖长度
} tree[MAXN * 8];

void build(int o, int l, int r) {
    tree[o].cover = 0;
    tree[o].len = 0;
    if (l == r) return;
    int mid = (l + r) / 2;
    build(o * 2, l, mid);
    build(o * 2 + 1, mid + 1, r);
}

void pushup(int o, int l, int r) {
    if (tree[o].cover > 0) {
        tree[o].len = ys[r + 1] - ys[l];
    } else if (l == r) {
        tree[o].len = 0;
    } else {
        tree[o].len = tree[o * 2].len + tree[o * 2 + 1].len;
    }
}

void update(int o, int l, int r, int L, int R, int v) {
    if (L <= l && r <= R) {
        tree[o].cover += v;
        pushup(o, l, r);
        return;
    }
    int mid = (l + r) / 2;
    if (L <= mid) update(o * 2, l, mid, L, R, v);
    if (R > mid) update(o * 2 + 1, mid + 1, r, L, R, v);
    pushup(o, l, r);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int caseNo = 0;
    while (cin >> n && n) {
        yCnt = 0;
        for (int i = 0; i < n; i++) {
            double x1, y1, x2, y2;
            cin >> x1 >> y1 >> x2 >> y2;
            ev[i * 2] = {x1, y1, y2, 1};
            ev[i * 2 + 1] = {x2, y1, y2, -1};
            ys[yCnt++] = y1;
            ys[yCnt++] = y2;
        }
        // 离散化 y
        sort(ys, ys + yCnt);
        yCnt = unique(ys, ys + yCnt) - ys;
        // 排序事件
        sort(ev, ev + 2 * n);
        // 建树
        build(1, 0, yCnt - 2);
        double ans = 0;
        for (int i = 0; i < 2 * n; i++) {
            if (i > 0) {
                double dx = ev[i].x - ev[i - 1].x;
                ans += dx * tree[1].len;
            }
            // 找到 y1, y2 的离散化下标
            int l = lower_bound(ys, ys + yCnt, ev[i].y1) - ys;
            int r = lower_bound(ys, ys + yCnt, ev[i].y2) - ys;
            // 区间 [l, r-1] 对应原始 [y1, y2]
            update(1, 0, yCnt - 2, l, r - 1, ev[i].kind);
        }
        cout << "Test case #" << ++caseNo << "\n";
        cout << "Total explored area: " << fixed << setprecision(2) << ans << "\n\n";
    }
    return 0;
}
// 输入：
// 2
// 10 10 20 20
// 15 15 25 25
// 0
// 输出：
// Test case #1
// Total explored area: 175.00
// 说明：矩形1面积=10×10=100，矩形2面积=10×10=100，
//       重叠区域[15,20]×[15,20]=5×5=25，并集=100+100-25=175。
```

!!! tip "离散化下标对应关系"
    - y 数组 `ys[0..yCnt-1]` 存储所有去重后的 y 坐标。
    - 线段树维护的是"基本区间" $[ys[i], ys[i+1]]$，共 $yCnt - 1$ 个。
    - 线段树下标 $i$ 对应原始区间 $[ys[i], ys[i+1]]$。
    - 更新 $[y_1, y_2]$ 时，找到 $l$、$r$ 使得 $ys[l] = y_1$、$ys[r] = y_2$，更新区间 $[l, r-1]$。

### 3.6 扫描线求矩形周长并（简介）

求多个矩形并集的**周长**，思路类似面积并：

- 仍按 $x$ 排序事件。
- 维护 y 轴被覆盖长度的同时，统计"覆盖长度变化量"——每两个相邻事件之间，y 方向覆盖长度的差值就是该段竖直边的贡献。
- 加上每段水平边（事件处 x 的变化）的贡献。

周长并的代码比面积并略复杂，需要在更新线段树时记录"覆盖长度的变化量"。CSP-S 中周长并考察较少，掌握面积并即可。

### 3.7 复杂度

| 维度 | 复杂度 | 说明 |
|------|:------:|------|
| 时间 | $O(n \log n)$ | 排序 $O(n \log n)$ + 线段树 $n$ 次更新每次 $O(\log n)$ |
| 空间 | $O(n)$ | 离散化数组 + 线段树 |

---

## 常见误区

### 误区一：Manacher 忘记插入分隔符

```cpp
// ✗ 错误：直接在原串上跑，偶长度回文无法处理
for (int i = 0; i < n; i++) {
    P[i] = 1;
    while (i - P[i] >= 0 && i + P[i] < n && s[i - P[i]] == s[i + P[i]]) P[i]++;
}

// ✓ 正确：先插入 # 统一奇偶
t[len++] = '#';
for (int i = 0; i < n; i++) {
    t[len++] = s[i];
    t[len++] = '#';
}
```

### 误区二：Manacher 中对称点计算错误

```cpp
// ✗ 错误：对称点公式记错
int j = 2 * i - mid;       // 方向反了

// ✓ 正确：j 关于 mid 的对称点
int j = 2 * mid - i;       // i + j = 2 * mid
```

### 误区三：Manacher 中 P[i] 初始化错误

```cpp
// ✗ 错误：i < mr 时直接 P[i] = P[j]
if (i < mr) P[i] = P[2 * mid - i];

// ✓ 正确：取 min，避免超过右边界
if (i < mr) P[i] = min(P[2 * mid - i], mr - i);
```

`P[j]` 可能超过 `mr - i`，此时对称回文的一部分超出已知右边界，无法保证对称性，必须从 `mr - i` 开始扩展。

### 误区四：Manacher 忘记更新 mr 和 mid

```cpp
// ✗ 错误：扩展后不更新右边界
while (...) P[i]++;
// 缺少更新 mr、mid 的代码

// ✓ 正确：扩展后更新
if (i + P[i] - 1 > mr) {
    mr = i + P[i] - 1;
    mid = i;
}
```

不更新 `mr` 会退化回 $O(n^2)$。

### 误区五：扫描线忘记离散化

```cpp
// ✗ 错误：y 坐标范围 10^9，直接建线段树会爆空间
// ✓ 正确：先离散化
sort(ys, ys + yCnt);
yCnt = unique(ys, ys + yCnt) - ys;
```

### 误区六：扫描线 cover 标记误用 lazy 下传

```cpp
// ✗ 错误：把 cover 当 lazy 标记下传给子节点
void pushdown(int o) {
    tree[o*2].cover += tree[o].cover;
    tree[o*2+1].cover += tree[o].cover;
    tree[o].cover = 0;          // 扫描线中这是错的！
}

// ✓ 正确：cover 不下传，只在 pushup 时按 cover>0 判断
void pushup(int o, int l, int r) {
    if (tree[o].cover > 0) {
        tree[o].len = ys[r+1] - ys[l];
    } else if (l == r) {
        tree[o].len = 0;
    } else {
        tree[o].len = tree[o*2].len + tree[o*2+1].len;
    }
}
```

!!! danger "扫描线线段树的特殊性"
    扫描线的线段树与普通区间更新线段树不同：`cover` 是"该区间被完整覆盖的次数"，**成对加减**保证平衡，无需下传。若误用 lazy 下传，会破坏"成对"性质导致错误。

### 误区七：扫描线离散化下标对应错误

```cpp
// ✗ 错误：更新区间 [l, r] 对应原始 [ys[l], ys[r]]
update(1, 0, yCnt-1, l, r, v);   // 范围搞错

// ✓ 正确：更新 [l, r-1] 对应 [ys[l], ys[r]]
update(1, 0, yCnt-2, l, r-1, v);
```

线段树的第 $i$ 个叶子对应基本区间 $[ys[i], ys[i+1]]$，所以覆盖 $[y_1, y_2]$（其中 $ys[l]=y_1$、$ys[r]=y_2$）对应更新线段树下标 $[l, r-1]$。

---

## 典型例题

### 例题一：【P3805】Manacher 模板

**题目链接**：https://www.luogu.com.cn/problem/P3805

**题意**：给出一个字符串 $s$（长度 $\le 1.1 \times 10^7$），求最长回文子串的长度。

**思路**：

1. 字符串长度达 $10^7$，必须用 $O(n)$ 的 Manacher 算法。
2. 插入分隔符 `#` 统一奇偶长度。
3. 维护 `mr`、`mid`，用对称性加速计算 `P` 数组。
4. 答案 $= \max(P[i]) - 1$。

**完整 AC 代码**：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 11000005;
char t[MAXN * 2];
int P[MAXN * 2];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    string s;
    cin >> s;
    int n = s.size();
    int len = 0;
    t[len++] = '#';
    for (int i = 0; i < n; i++) {
        t[len++] = s[i];
        t[len++] = '#';
    }
    t[len] = '\0';
    int mr = 0, mid = 0, ans = 0;
    for (int i = 0; i < len; i++) {
        if (i < mr) {
            P[i] = min(P[2 * mid - i], mr - i);
        } else {
            P[i] = 1;
        }
        while (i - P[i] >= 0 && i + P[i] < len && t[i - P[i]] == t[i + P[i]]) {
            P[i]++;
        }
        if (i + P[i] - 1 > mr) {
            mr = i + P[i] - 1;
            mid = i;
        }
        ans = max(ans, P[i] - 1);
    }
    cout << ans << "\n";
    return 0;
}
// 输入：ababa
// 输出：5
// 输入：aaaa
// 输出：4
// 输入：abc
// 输出：1
```

**关键点**：

- `mr` 必须更新，否则退化 $O(n^2)$。
- `P[i] = min(P[2*mid-i], mr-i)` 是加速的核心。
- 注意字符串长度上限，数组开够。

---

### 例题二：【P5490】扫描线模板（矩形面积并）

**题目链接**：https://www.luogu.com.cn/problem/P5490

**题意**：平面上有 $n$ 个矩形（$n \le 10^5$，坐标范围 $10^9$），求它们并集的面积。

**思路**：

1. 每个矩形拆成"加入"和"移除"两个事件。
2. 对 y 坐标离散化。
3. 用线段树维护 y 轴上每个基本区间的"覆盖次数"和"覆盖长度"。
4. 扫描线按 $x$ 顺序处理事件，累计相邻事件间的面积贡献。

**完整 AC 代码**：

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAXN = 200005;
int n;
ll ys[MAXN * 2];
int yCnt = 0;

struct Event {
    ll x, y1, y2;
    int kind;
    bool operator<(const Event &o) const {
        return x < o.x;
    }
} ev[MAXN * 2];

struct Node {
    int cover;
    ll len;
} tree[MAXN * 8];

void build(int o, int l, int r) {
    tree[o].cover = 0;
    tree[o].len = 0;
    if (l == r) return;
    int mid = (l + r) / 2;
    build(o * 2, l, mid);
    build(o * 2 + 1, mid + 1, r);
}

void pushup(int o, int l, int r) {
    if (tree[o].cover > 0) {
        tree[o].len = ys[r + 1] - ys[l];
    } else if (l == r) {
        tree[o].len = 0;
    } else {
        tree[o].len = tree[o * 2].len + tree[o * 2 + 1].len;
    }
}

void update(int o, int l, int r, int L, int R, int v) {
    if (L <= l && r <= R) {
        tree[o].cover += v;
        pushup(o, l, r);
        return;
    }
    int mid = (l + r) / 2;
    if (L <= mid) update(o * 2, l, mid, L, R, v);
    if (R > mid) update(o * 2 + 1, mid + 1, r, L, R, v);
    pushup(o, l, r);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n;
    yCnt = 0;
    for (int i = 0; i < n; i++) {
        ll x1, y1, x2, y2;
        cin >> x1 >> y1 >> x2 >> y2;
        ev[i * 2] = {x1, y1, y2, 1};
        ev[i * 2 + 1] = {x2, y1, y2, -1};
        ys[yCnt++] = y1;
        ys[yCnt++] = y2;
    }
    sort(ys, ys + yCnt);
    yCnt = unique(ys, ys + yCnt) - ys;
    sort(ev, ev + 2 * n);
    build(1, 0, yCnt - 2);
    ll ans = 0;
    for (int i = 0; i < 2 * n; i++) {
        if (i > 0) {
            ll dx = ev[i].x - ev[i - 1].x;
            ans += dx * tree[1].len;
        }
        int l = lower_bound(ys, ys + yCnt, ev[i].y1) - ys;
        int r = lower_bound(ys, ys + yCnt, ev[i].y2) - ys;
        update(1, 0, yCnt - 2, l, r - 1, ev[i].kind);
    }
    cout << ans << "\n";
    return 0;
}
// 输入：
// 2
// 1 1 3 3
// 2 2 4 4
// 输出：
// 7
```

**关键点**：

- 坐标范围 $10^9$，必须离散化。
- `cover` 不下传，只在 `pushup` 时判断。
- 注意 P5490 中坐标是整数（ll），而 POJ 1151（经典面积并）是浮点数。
- 答案可能很大，用 `long long`。

!!! tip "扫描线代码模板记忆"
    1. 拆事件（每个矩形 2 个）
    2. 离散化 y
    3. 排序事件按 x
    4. 建线段树
    5. 依次处理事件，累加 `dx * tree[1].len`
    6. 更新线段树区间 `[l, r-1]`

---

## 📖 本章总结

- **回文串基础**
  - 定义：正读反读相同
  - 奇长度（字符为中心）、偶长度（间隙为中心）
  - 朴素算法 $O(n^2)$，浪费对称信息
- **Manacher 算法**
  - 核心思想：利用已知回文对称性加速
  - 插入 `#` 统一奇偶长度
  - `P[i]`：以 `t[i]` 为中心的最长回文半径
  - 加速：`P[i] = min(P[2*mid-i], mr-i)`（`i < mr` 时）
  - 更新 `mr`、`mid` 保证线性
  - 答案：`max(P[i]) - 1`
  - 复杂度：$O(n)$
- **扫描线算法**
  - 基本思想：一条线扫过平面，维护状态
  - 矩形面积并：拆事件 + 离散化 + 线段树
  - 离散化：y 坐标排序去重
  - 线段树：`cover`（覆盖次数，不下传）+ `len`（覆盖长度）
  - `pushup`：`cover>0` 则整段覆盖，否则子树求和
  - 复杂度：$O(n \log n)$
  - 周长并：类似面积并，额外统计覆盖长度变化量
- **常见误区**
  - Manacher 忘记插 `#`
  - 对称点公式 `2*mid-i` 记反
  - `P[i]` 初始化未取 `min(mr-i)`
  - 忘记更新 `mr`、`mid`
  - 扫描线忘记离散化
  - `cover` 误用 lazy 下传
  - 离散化区间下标对应错误（`[l, r-1]` 而非 `[l, r]`）
- **典型例题**
  - P3805 Manacher 模板：求最长回文子串
  - P5490 扫描线模板：矩形面积并

### 自测清单（合上书自检）

!!! tip "能做到以下几点，才算真正掌握本章"
    - [ ] 说出回文串的定义，解释朴素算法为何 $O(n^2)$
    - [ ] 默写 Manacher 算法，包括插 `#`、`P` 数组、`mr`/`mid` 更新
    - [ ] 说明为何插 `#` 能统一奇偶长度
    - [ ] 推导 Manacher 的 $O(n)$ 复杂度（`mr` 单调不减）
    - [ ] 描述扫描线求矩形面积并的流程
    - [ ] 默写扫描线 + 线段树代码，说明 `cover` 为何不下传
    - [ ] 解释离散化下标对应：线段树下标 $i$ 对应 $[ys[i], ys[i+1]]$
    - [ ] 用 Manacher 解决 P3805
    - [ ] 用扫描线解决 P5490

### 核心速查表

| 操作 | 代码/公式 | 说明 |
|------|----------|------|
| 插入分隔符 | `t = #s[0]#s[1]#...#s[n-1]#` | 长度 $2n+1$ |
| P 数组初始化 | `P[i]=min(P[2*mid-i], mr-i)` | `i < mr` 时 |
| P 数组扩展 | `while(t[i-P[i]]==t[i+P[i]]) P[i]++` | 注意边界 |
| 更新右边界 | `if(i+P[i]-1>mr){mr=i+P[i]-1;mid=i;}` | 必须更新 |
| 最长回文长度 | `ans = max(P[i]) - 1` | 原串长度 |
| 扫描线拆事件 | 每个矩形 2 个事件（+1/-1） | 按 x 排序 |
| y 离散化 | `sort + unique` | 基本区间 $[ys[i],ys[i+1]]$ |
| 线段树 cover | `tree[o].cover += v` | 不下传 |
| 线段树 pushup | `cover>0 → len=ys[r+1]-ys[l]` | 整段覆盖 |
| 更新区间 | `update(l, r-1, v)` | 对应 $[y_1, y_2]$ |
| 面积累加 | `ans += dx * tree[1].len` | 相邻事件间 |

### 知识衔接

本章是考纲补强卷的第四章，与前后章节的联系如下：

- **前置知识**：第24章Trie树与KMP（字符串匹配思想是Manacher的铺垫）、第52章线段树（扫描线算法配合线段树实现区间维护）
- **本章定位**：讲解线性时间求最长回文子串的Manacher算法与矩形面积并的扫描线算法，是2025年CSP-S新增考纲的重点内容
- **后续发展**：本章"利用已有信息加速计算"的思想为第55章单调栈与单调队列铺路，后者同样强调维护单调性以加速查询

!!! info "下一章预告"
    本章我们学习了 Manacher 与扫描线两个利用"已有信息加速计算"的经典算法。下一章将转向另一类"维护单调性"的利器——**单调栈与单调队列**。单调栈用于求"每个元素左侧/右侧第一个比它大/小的元素"，单调队列用于在滑动窗口中维护最值，它们都是 CSP-S 的高频模板，也是优化 DP 的重要工具。
