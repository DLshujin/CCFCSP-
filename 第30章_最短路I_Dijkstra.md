# 第30章 最短路I：Dijkstra

> 最短路问题是图论中最经典、最重要的问题之一。在 CSP-S 提高级中，Dijkstra 算法是必考内容，掌握它就等于拿下了图论的一座大山。

!!! info "本章内容"
    本章聚焦**单源最短路 + 非负权图**，系统讲解：最短路问题概述、Dijkstra 算法核心思想（贪心 + 松弛）、朴素版（邻接矩阵，$O(n^2)$）与堆优化版（邻接表 + priority_queue，$O(m \log n)$）的完整实现、两种版本对比与选择策略、最短路计数等扩展应用。所有代码统一使用 `#include <bits/stdc++.h>` 与邻接表建图。

!!! warning "本章定位"
    Dijkstra 是 CSP-S 提高级的必考算法。堆优化版是处理 $n \leq 10^5$ 稀疏图的首选方案，SPFA 在标准版题目中会被卡。掌握 Dijkstra 的"松弛 + 贪心"思想，也是后续学习 SPFA、Floyd 等最短路算法的基础。

---

## 📌 学习目标

学完本章后，你应当能够：

1. **准确说出**最短路问题的定义，区分单源最短路（SSSP）与多源最短路（APSP）
2. **解释** Dijkstra 算法的贪心正确性——为什么边权非负时，当前 dist 最小的未访问点可以直接确定
3. **默写**朴素 Dijkstra（邻接矩阵）的完整代码，包括 `memset(dist, 0x3f, ...)` 初始化
4. **默写**堆优化 Dijkstra（邻接表 + priority_queue 小根堆）的完整代码
5. **理解** `vis` 数组在堆优化版中的作用（惰性删除），并说明不判 `vis` 会导致复杂度退化
6. **避开**常见误区：负权图不能用 Dijkstra、INF 不能设 INT_MAX、priority_queue 默认是大根堆、无向图需加双向边

---

## 💡 生活类比

### 类比一：水波纹扩散

想象你往平静的湖面投下一颗石子，**水波纹会从落点向外一圈圈扩散**，先到达近处，再到达远处。Dijkstra 算法就像这水波纹：

- 每次从"已确定最短路的点集"中，向外延伸一条边
- 选择**当前离起点最近**的未访问点，加入点集
- 用新加入的点去"松弛"它的邻居（试试能不能让邻居更近）

这种"**每次选最近的**"策略，本质上是**贪心**思想。

### 类比二：导航地图

打开手机导航，输入起点和终点，地图会告诉你最短路径。这背后的核心算法之一就是 Dijkstra。导航规划的本质就是：在带权图（路口是点，道路是边，长度是权值）上，求两点间的最短路。

---

## 一、最短路问题概述

### 1.1 什么是最短路

给定一张带权图 $G=(V, E)$，其中 $V$ 是顶点集合，$E$ 是边集合，每条边 $e$ 有一个权值 $w(e)$（也称长度/代价）。**最短路问题**就是：求从起点 $s$ 到终点 $t$ 的路径中，边权之和最小的那一条。

```
        2         3
   A -------- B ------- C
   |          |          \
   |1         |4          \ 1
   |          |            \
   D -------- E ----------- F
        5         2
```

例如上图中，从 A 到 F 的最短路是 A→B→C→F（$2+3+1=6$），这是最短路径。而 A→B→E→F（$2+4+2=8$）与 A→D→E→F（$1+5+2=8$）虽然也通，但长度更大。

### 1.2 问题分类

| 分类维度 | 类型 | 说明 |
|---------|------|------|
| 按起点数量 | **单源最短路 (SSSP)** | 求一个源点到其他所有点的最短路 |
|              | **多源最短路 (APSP)** | 求任意两点之间的最短路 |
| 按边权符号 | **非负权图** | 所有边权 $\geq 0$，可用 Dijkstra |
|            | **负权图** | 存在负权边，需用 Bellman-Ford / SPFA |
| 按图方向 | **有向图** | 边有方向，$u \to v$ 不代表 $v \to u$ |
|          | **无向图** | 边可双向通行，相当于两条有向边 |

!!! info "本章范围"
    本章聚焦**单源最短路 + 非负权图**，这是 Dijkstra 算法的"主场"。负权图和多源最短路（Floyd）将在后续章节讲解。

---

## 二、Dijkstra 算法核心思想

### 2.1 两个核心概念

#### 1. 松弛操作（Relaxation）

这是最短路的灵魂。设 $dist[u]$ 表示当前已知的从源点 $s$ 到 $u$ 的最短距离。对于边 $u \xrightarrow{w} v$：

```
如果 dist[u] + w < dist[v]:
    dist[v] = dist[u] + w   // 找到了更短的路径，更新它
```

形象地说，"把 $v$ 的距离**松一松**，看能不能更近"。

#### 2. 已确定集合 S

维护一个集合 $S$，其中所有点的最短路**已经被最终确定**。每次从不在 $S$ 中的点里，选 $dist$ 最小的那个加入 $S$。

!!! warning "为什么贪心是对的？"
    因为边权非负。当 $u$ 是当前 $dist$ 最小的未访问点时，任何其他路径想要到达 $u$，都必须经过某个 $dist$ 更大的点，加上非负权后只会更大。所以 $dist[u]$ 已经是最优值，可以放心确定。

### 2.2 算法步骤

```
1. 初始化：dist[s] = 0，其余 dist[i] = +∞，集合 S = ∅
2. 重复 n 次：
   a. 从未加入 S 的点中，选 dist 最小的点 u
   b. 将 u 加入 S（标记已确定）
   c. 对 u 的每条出边 (u, v, w)，执行松弛：dist[v] = min(dist[v], dist[u]+w)
3. 结束，dist 数组即为答案
```

### 2.3 流程示意

```
源点 s=1，图如下（括号内为边权）：

    (2)        (3)
  1 ----> 2 ----> 4
  |       ^       |
 (4)|   (1)|   (5)|
  v       |       v
  3 ----> 5 ----> 6
    (6)        (2)

Step1: 选 dist[1]=0 的点 1，松弛 2,3
       dist = [0, 2, 4, ∞, ∞, ∞]
Step2: 选 dist=2 的点 2，松弛 4
       dist = [0, 2, 4, 5, ∞, ∞]
Step3: 选 dist=4 的点 3，松弛 5
       dist = [0, 2, 4, 5, 10, ∞]
Step4: 选 dist=5 的点 4，松弛 6
       dist = [0, 2, 4, 5, 10, 10]
Step5: 选 dist=10 的点 5，松弛 6 → 10+2=12 > 10，不更新
Step6: 选 dist=10 的点 6，结束

最终答案：dist = [0, 2, 4, 5, 10, 10]
```

---

## 三、朴素 Dijkstra（邻接矩阵）

### 3.1 适用场景

**稠密图**（边数 $m$ 接近 $n^2$），用邻接矩阵存储，时间复杂度 $O(n^2)$。

### 3.2 完整代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1005;
const int INF = 0x3f3f3f3f;

int n, m, s;
int g[MAXN][MAXN];     // 邻接矩阵
int dist[MAXN];        // 最短路数组
bool vis[MAXN];        // 是否已确定

void dijkstra(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, false, sizeof(vis));
    dist[s] = 0;
    for (int i = 1; i <= n; i++) {
        // 找当前 dist 最小的未访问点
        int u = -1, minv = INF;
        for (int j = 1; j <= n; j++) {
            if (!vis[j] && dist[j] < minv) {
                minv = dist[j];
                u = j;
            }
        }
        if (u == -1) break;   // 剩余点不可达
        vis[u] = true;
        // 用 u 松弛所有邻居
        for (int v = 1; v <= n; v++) {
            if (!vis[v] && g[u][v] < INF) {
                dist[v] = min(dist[v], dist[u] + g[u][v]);
            }
        }
    }
}

int main() {
    cin >> n >> m >> s;
    memset(g, 0x3f, sizeof(g));
    for (int i = 1; i <= n; i++) g[i][i] = 0;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        // 处理重边：保留最短的
        if (w < g[u][v]) g[u][v] = w;
    }
    dijkstra(s);
    for (int i = 1; i <= n; i++) {
        cout << dist[i] << " ";
    }
    cout << endl;
    return 0;
}
/*
输入：
4 5 1
1 2 2
1 3 4
2 3 1
2 4 7
3 4 3

预期输出：
0 2 3 6
*/
```

### 3.3 关键细节

| 细节 | 说明 |
|------|------|
| `INF` 取值 | 用 `0x3f3f3f3f`，满足 `INF + INF` 不溢出且大于任何合理路径长度 |
| `memset(dist, 0x3f, ...)` | 把每个 int 设为 `0x3f3f3f3f`，比 `INF=1e9` 更方便 |
| 重边处理 | 邻接矩阵取 `min`，只保留最短边 |
| 自环 | `g[i][i]=0`，不影响结果 |
| 提前终止 | 当 `u==-1`（找不到可达点）时 `break` |

!!! tip "为什么用 0x3f3f3f3f？"
    - 它约等于 $10^9$，足够大
    - 两个相加约 $2\times10^9$，不超 `int` 范围
    - `memset` 按字节填充，`0x3f` 每个字节都一样，可以一次性设好整个数组

---

## 四、堆优化 Dijkstra（邻接表 + 优先队列）

### 4.1 适用场景

**稀疏图**（边数 $m$ 远小于 $n^2$），用邻接表 + 小根堆，时间复杂度 $O((n+m)\log n) \approx O(m \log n)$。

### 4.2 核心优化思路

朴素版每轮要 $O(n)$ 扫描找最小点，太慢。用**小根堆**（priority_queue）维护候选点，每次 $O(\log n)$ 取出最小者，总时间降到 $O(m \log n)$。

### 4.3 完整代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e5 + 5;
const int INF = 0x3f3f3f3f;

int n, m, s;
int dist[MAXN];
bool vis[MAXN];

struct Edge {
    int to, w;
};
vector<Edge> G[MAXN];

void dijkstra(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, false, sizeof(vis));
    dist[s] = 0;
    // 小根堆：pair<距离, 节点编号>
    // 距离放在 first，堆自动按距离排序
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, s});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (vis[u]) continue;     // 已确定，跳过（惰性删除）
        vis[u] = true;
        for (auto &e : G[u]) {
            int v = e.to, w = e.w;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        G[u].push_back({v, w});
        // 无向图加：G[v].push_back({u, w});
    }
    dijkstra(s);
    for (int i = 1; i <= n; i++) {
        cout << dist[i] << " ";
    }
    cout << endl;
    return 0;
}
/*
输入：
4 5 1
1 2 2
1 3 4
2 3 1
2 4 7
3 4 3

预期输出：
0 2 3 6
*/
```

### 4.4 关键细节

!!! warning "priority_queue 默认是大根堆！"
    C++ 默认 `priority_queue<int>` 是**大根堆**（顶部最大）。要实现小根堆，必须写：
    ```cpp
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    ```
    - 第二个参数 `vector<...>` 是底层容器
    - 第三个参数 `greater<...>` 是比较器，让小的在前

!!! tip "为什么用 pair<int,int>？"
    `pair` 默认按 `first` 优先比较，我们存 `{距离, 节点}`，堆就会按距离排序。简单高效，无需自定义结构体。

!!! warning "vis 数组的作用：惰性删除"
    一个节点可能被多次 push 进堆（每次松弛都 push 一次）。取出时如果已 `vis`，直接跳过，避免重复处理。这是**惰性删除**技巧——不主动从堆中删除旧数据，而是在取出时判断。

### 4.5 复杂度分析

| 操作 | 次数 | 单次复杂度 |
|------|------|-----------|
| 取堆顶 | 最多 $m$ 次（每条边可能 push 一次） | $O(\log n)$ |
| 松弛 push | 最多 $m$ 次 | $O(\log n)$ |
| 总计 | | $O(m \log n)$ |

---

## 五、两种实现对比

| 对比项 | 朴素 Dijkstra | 堆优化 Dijkstra |
|-------|--------------|----------------|
| 存储结构 | 邻接矩阵 `g[][]` | 邻接表 `vector<Edge>` |
| 时间复杂度 | $O(n^2)$ | $O(m \log n)$ |
| 空间复杂度 | $O(n^2)$ | $O(n + m)$ |
| 适用图类型 | **稠密图**（$m \approx n^2$） | **稀疏图**（$m \ll n^2$） |
| 代码量 | 短，易写 | 略长，涉及 STL |
| 找最小点方式 | $O(n)$ 线性扫描 | $O(\log n)$ 堆操作 |
| 常数 | 小 | 略大（堆操作 + STL） |
| 推荐场景 | $n \leq 2000$ 且图稠密 | $n \leq 10^5$ 的稀疏图 |

!!! info "选择建议"
    - 提高级题目 $n \leq 10^5$，**绝大多数用堆优化版**
    - 若题目 $n$ 很小（如 $n \leq 500$）且图为完全图，朴素版更快
    - 不确定时，**默认写堆优化版**

---

## 六、常见误区

### 误区 1：负权图使用 Dijkstra

!!! danger "错误"
    ```cpp
    // 图中有边权为 -5
    if (dist[u] + w < dist[v]) dist[v] = dist[u] + w;
    ```

**为什么错？** Dijkstra 的贪心基础是"已确定点的 dist 不会再变"。但若有负权边，后面可能通过负边"反悔"出更短路径，贪心失效。

**反例：**

考虑下图（含负权边 $3 \to 2$，权 $-3$）：

```
    1 --(1)--> 2
     \         ^
    (2)\       |(-3)
       v       |
       3 ------+
```

Dijkstra 从 1 出发：先确定 $\text{dist}[1]=0$，松弛得 $\text{dist}[2]=1$（走 $1\to 2$）、$\text{dist}[3]=2$（走 $1\to 3$）。接着选当前最小未确定点 2，把 $\text{dist}[2]=1$ **确定为最终值**。但实际最短路是 $1\to 3\to 2 = 2+(-3)=-1$，比 1 更短！Dijkstra 给出了错误答案。**结论：负权图请用 SPFA / Bellman-Ford。**

### 误区 2：dist 初始化为 INT_MAX

```cpp
const int INF = INT_MAX;     // 危险！
dist[u] + w                  // 溢出，结果未定义
```

**正确做法：** 用 `0x3f3f3f3f`，保证相加不溢出。

### 误区 3：忘记处理重边

邻接矩阵版需 `g[u][v] = min(g[u][v], w)`；邻接表版会全部存入，松弛时自然取最小，但**多浪费空间**。

### 误区 4：堆中存的是过时数据，不判 vis

```cpp
// 错误写法：从堆顶取出后直接处理，不判 vis
auto [d, u] = pq.top(); pq.pop();
// 应该加：if (vis[u]) continue;
```

不判 `vis` 会导致同一节点被多次松弛，复杂度退化。

### 误区 5：pair 第二维存节点时弄反顺序

```cpp
pq.push({u, dist[u]});   // 错！堆按 first 排序，应让距离在前
pq.push({dist[u], u});   // 对
```

### 误区 6：无向图只加一条边

无向图必须 `G[u].push_back({v,w}); G[v].push_back({u,w});` 加两条。

---

## 七、核心操作速查表

| 操作 | 朴素版 | 堆优化版 |
|------|--------|---------|
| 初始化 | `memset(dist, 0x3f, sizeof(dist))` | 同左 |
| 起点设 0 | `dist[s] = 0` | `dist[s] = 0; pq.push({0, s})` |
| 找最小点 | $O(n)$ 循环扫描 | `pq.top(); pq.pop()` |
| 标记已确定 | `vis[u] = true` | `if(vis[u]) continue; vis[u]=true` |
| 松弛 | `dist[v] = min(dist[v], dist[u]+g[u][v])` | `if(dist[u]+w<dist[v]){dist[v]=dist[u]+w; pq.push({dist[v],v})}` |
| 终止条件 | 循环 $n$ 次或 `u==-1` | 堆空 |

---

## 八、典型例题

### 例题 1：[P3371] 单源最短路径（弱化版）

**题目描述：** 给定 $n$ 个点 $m$ 条有向边，求从起点 $s$ 到每个点的最短路。$n \leq 10^4, m \leq 5\times10^5, w \leq 10^6$。

**数据特点：** 弱化版允许 SPFA，但堆优化 Dijkstra 同样可过。

**思路：** 堆优化 Dijkstra 模板题。注意输出格式——不可达点输出 `2147483647`。

**参考代码：**

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e4 + 5;
const int INF = 0x3f3f3f3f;

int n, m, s;
int dist[MAXN];
bool vis[MAXN];
struct Edge { int to, w; };
vector<Edge> G[MAXN];

void dijkstra(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, false, sizeof(vis));
    dist[s] = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, s});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (auto &e : G[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                pq.push({dist[e.to], e.to});
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        G[u].push_back({v, w});
    }
    dijkstra(s);
    for (int i = 1; i <= n; i++) {
        // P3371 特殊要求：不可达点输出 2147483647
        cout << (dist[i] >= INF ? (int)2147483647 : dist[i]);
        cout << (i == n ? '\n' : ' ');
    }
    return 0;
}
/*
输入：
4 6 1
1 2 2
2 3 2
2 4 1
1 3 5
3 4 3
1 4 4

预期输出：
0 2 4 3
*/
```

!!! warning "P3371 输出坑点"
    不可达点要输出 `2147483647`，不是 `0x3f3f3f3f`。注意转换。

---

### 例题 2：[P4779] 单源最短路径（标准版）

**题目描述：** 同 P3371，但数据加强：$n \leq 10^5, m \leq 2\times10^5$，且**卡 SPFA**。

**核心：** 必须用堆优化 Dijkstra，SPFA 会被 TLE。

**参考代码：**

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e5 + 5;
const int INF = 0x3f3f3f3f;

int n, m, s;
int dist[MAXN];
bool vis[MAXN];
struct Edge { int to, w; };
vector<Edge> G[MAXN];

void dijkstra(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, false, sizeof(vis));
    dist[s] = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, s});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (auto &e : G[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                pq.push({dist[e.to], e.to});
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        G[u].push_back({v, w});
    }
    dijkstra(s);
    for (int i = 1; i <= n; i++) {
        cout << dist[i] << (i == n ? '\n' : ' ');
    }
    return 0;
}
/*
输入：
4 6 1
1 2 2
2 3 2
2 4 1
1 3 5
3 4 3
1 4 4

预期输出：
0 2 4 3
*/
```

!!! tip "为什么 P4779 卡 SPFA？"
    SPFA 平均 $O(km)$（$k$ 为小常数），但最坏 $O(nm)$。出题人构造特殊数据（如网格图、菊花图）能让 SPFA 退化到 $O(nm)$ 而 TLE。Dijkstra 堆优化 $O(m\log n)$ 稳定可靠。

---

### 例题 3：[P1144] 最短路计数

**题目描述：** 给定无向无权图，求从 1 号点到每个点的最短路条数，模 100003。

**思路：** 在 Dijkstra 过程中，**额外维护一个 `cnt[]` 数组**记录到达每个点的最短路数目：

- 当 `dist[u] + 1 < dist[v]`：找到更短路，`cnt[v] = cnt[u]`
- 当 `dist[u] + 1 == dist[v]`：找到等长路，`cnt[v] = (cnt[v] + cnt[u]) % MOD`

**参考代码：**

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e6 + 5;
const int MOD = 100003;

int n, m;
int dist[MAXN];
int cnt[MAXN];
bool vis[MAXN];
vector<int> G[MAXN];

void dijkstra(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(vis, false, sizeof(vis));
    dist[s] = 0;
    cnt[s] = 1;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, s});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int v : G[u]) {
            if (dist[u] + 1 < dist[v]) {
                dist[v] = dist[u] + 1;
                cnt[v] = cnt[u];
                pq.push({dist[v], v});
            } else if (dist[u] + 1 == dist[v]) {
                cnt[v] = (cnt[v] + cnt[u]) % MOD;
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    dijkstra(1);
    for (int i = 1; i <= n; i++) {
        cout << cnt[i] << '\n';
    }
    return 0;
}
/*
输入：
5 7
1 2
1 3
2 4
3 4
2 3
4 5
4 5

预期输出：
1
1
1
2
4
*/
```

!!! tip "最短路计数的本质"
    利用 Dijkstra 的"逐点确定"性质：当 $u$ 被确定时，所有以 $u$ 为倒数第二个点的最短路都已就绪，直接累加即可。BFS 也能做（无权图等价于权为 1）。

---

## 九、调试技巧

### 9.1 打印 dist 数组

```cpp
for (int i = 1; i <= n; i++) cerr << dist[i] << " ";
cerr << endl;
```

### 9.2 检查图是否建对

```cpp
for (int u = 1; u <= n; u++) {
    cerr << u << ": ";
    for (auto &e : G[u]) cerr << "(" << e.to << "," << e.w << ") ";
    cerr << endl;
}
```

### 9.3 常见 Bug 排查

| 现象 | 可能原因 |
|------|---------|
| 全部输出 `0x3f3f3f3f` | 起点没 push / 图建反方向 |
| 输出负数或奇怪值 | 溢出，INF 设太大 |
| TLE | 用了朴素版 / 没判 vis 重复入堆 |
| WA 部分 | 重边未处理 / 无向图只加单向 |
| RE | 数组开小，MAXN 不够 |

---

## 十、拓展思考

### 10.1 反向图与最短路

求"每个点到终点 $t$ 的最短路"等价于在**反图**（将所有边反向）上跑一次从 $t$ 出发的 Dijkstra。常用于分层图、最短路 DAG 等场景。

### 10.2 输出具体路径

维护 `pre[v] = u`，松弛时记录前驱，最后从终点回溯：

```cpp
int path[MAXN], tot = 0;
for (int x = t; x != s; x = pre[x]) path[++tot] = x;
path[++tot] = s;
for (int i = tot; i >= 1; i--) cout << path[i] << " ";
```

### 10.3 k 短路（拓展）

求第 $k$ 短路需用 A* 算法，超出本章范围，留作后续章节内容。

---

### 知识衔接

本章是**最短路算法系列**的第一章，与前后章节的联系如下：

- **前置知识**：第22章堆与优先队列（堆优化的核心）、第29章拓扑排序（DAG上的最短路概念）、第26章图的存储与遍历（邻接表建图）
- **本章定位**：解决非负权图的单源最短路问题，是贪心思想在图论中的经典应用
- **后续发展**：为第31章Bellman-Ford与SPFA（处理负权）提供对比基础，堆优化Dijkstra是CSP-S图论的核心模板

## 十一、本章总结

### 核心要点

1. **Dijkstra 适用于非负权图的单源最短路问题**，负权图请用 SPFA/Bellman-Ford。
2. **核心思想**：贪心 + 松弛。每次选当前 dist 最小的未确定点加入集合 S，并用它松弛邻居。
3. **朴素版** $O(n^2)$，邻接矩阵，适合稠密图。
4. **堆优化版** $O(m \log n)$，邻接表 + priority_queue 小根堆，适合稀疏图，**提高级首选**。
5. **小根堆写法**：`priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>>`。
6. **INF 用 `0x3f3f3f3f`**，配合 `memset` 高效初始化。
7. **vis 数组**避免重复处理，是堆优化版的关键优化。

### 记忆口诀

> **"非负权，Dijkstra；稠密朴素稀疏堆；距离在前节点后；vis 一判省烦恼。"**

### 易错点回顾

- [ ] 负权图不能用 Dijkstra
- [ ] INF 不能用 INT_MAX（相加溢出）
- [ ] priority_queue 默认大根堆，要加 greater
- [ ] 堆中元素需判 vis 跳过
- [ ] 无向图加双向边
- [ ] 邻接矩阵处理重边取 min
- [ ] 不可达点输出格式（P3371 要 2147483647）

### 后续学习路线

| 章节 | 内容 |
|------|------|
| 第31章 | 最短路II：SPFA / Bellman-Ford（处理负权） |
| 第32章 | 最短路III：Floyd（多源最短路） |
| 第33章 | 最小生成树（Prim/Kruskal） |

---

## 十二、本节练习

| 题号 | 题目 | 难度 | 要点 |
|------|------|------|------|
| P3371 | 单源最短路径（弱化版） | ★★ | Dijkstra 模板 |
| P4779 | 单源最短路径（标准版） | ★★ | 堆优化，卡 SPFA |
| P1144 | 最短路计数 | ★★★ | 维护 cnt 数组 |
| P1462 | 通往奥格瑞玛的道路 | ★★★★ | 二分 + 最短路 |
| P1119 | 灾后重建 | ★★★★ | Floyd 时间顺序 |

!!! info "学习建议"
    - 先把 P3371 和 P4779 写到 AC，确保模板熟练
    - 再挑战 P1144，理解"在最短路框架上扩展状态"的思路
    - 最后尝试 P1462、P1119，体会最短路与其他算法的结合

---

> **下章预告：** 第31章将讲解 SPFA 与 Bellman-Ford，攻克负权图最短路难题，并讨论 SPFA 的优化与被卡原理。