# 第31章 最短路II：Bellman-Ford & SPFA

> 上一章我们学习了 Dijkstra 算法，它能高效解决**非负权图**的单源最短路问题。但现实世界并不总是"阳光明媚"——当图中出现**负权边**甚至**负权环**时，Dijkstra 的贪心策略就会失效。本章介绍两大能够处理负权边的算法：**Bellman-Ford** 与它的队列优化版本 **SPFA**。

!!! info "本章内容"
    - Dijkstra 的局限：为什么负权边和负权环会让贪心策略失效
    - Bellman-Ford 算法：n-1 轮全边松弛 + 第 n 轮判负环，时间 O(nm)
    - SPFA 算法：队列优化版 Bellman-Ford，期望 O(km)，最坏 O(nm)
    - SPFA 判负环：cnt[v] >= n 或入队次数超过 n-1
    - 三种最短路算法综合对比与选择决策树
    - 常见误区：SPFA 被卡、负环判断、INF 溢出等

---

## 📌 学习目标

学完本章后，你应当能够：

1. **解释**为什么 Dijkstra 在负权图上失效，并画出反例
2. **默写** Bellman-Ford 的 n-1 轮松弛 + 第 n 轮判负环模板
3. **默写** SPFA 的队列实现，并说明 `in_queue` 数组的作用
4. **区分**"判全图负环"与"判源点可达负环"的入队策略
5. **说出** SPFA 最坏退化 O(nm) 的原因及典型卡 SPFA 的图结构
6. **根据**题意在 Dijkstra / Bellman-Ford / SPFA 三者中做出正确选择

---

## 💡 生活类比

### 类比一：反复检查的质检员

Bellman-Ford 像一位"反复检查每条边能不能更短"的质检员。他不贪心，而是老老实实地把**所有边检查一遍又一遍**，直到没有距离能被更新为止。这就像全公司每天早上统一开会通报进度——虽然慢，但保证不漏掉任何一条变短的可能。

### 类比二：按需通知的同事

SPFA 则像"谁有新消息就通知相关同事"。只有当某个点的距离被更新了，它的邻居才可能因此变短。所以 SPFA 只把"刚被更新"的点入队，让它的邻居重新尝试松弛——**按需传递，效率更高**。

!!! tip "一句话记忆"
    Bellman-Ford 是"全量扫描"的稳重型，SPFA 是"按需通知"的敏捷型。两者都能处理负权边，但 SPFA 在竞赛中可能被特殊数据卡到退化。

---

## 第一节 为什么需要 Bellman-Ford？

### 1.1 Dijkstra 的局限

回顾 Dijkstra 的核心假设：**已确定最短路的点不会被后续更新**。这依赖"边权非负"——因为新路径只能越走越长。

一旦存在负权边，这个假设就崩塌了：

```
        2
   A ---------> B
   |            ^
   |            |
 1 |            | -3
   v            |
   C ----------+
        5
```

Dijkstra 从 A 出发会先选 C（距离 1），认为 C 已确定；但实际上 A→B→C 的距离是 `2 + (-3) = -1 < 1`，C 应该被更新。贪心策略出错。

!!! warning "Dijkstra 失效条件"
    只要图中存在**任意一条负权边**，Dijkstra 就**可能**给出错误答案。注意是"可能"而非"一定"——某些负权图 Dijkstra 仍能蒙对，但不能保证正确性。

### 1.2 负权环的灾难

更糟糕的情况是**负权环**（权值和为负的环）。绕着负环转一圈，距离就会减少；转无数圈，距离趋于负无穷。

```
     -1
  A -------> B
  ^          |
  |          |
  |  2       | -2
  |          v
  D <------- C
        1
```

环 A→B→C→D→A 的权值和 = `-1 + (-2) + 1 + 2 = 0`，不是负环。
若改为 A→B = -3，则环权和 = `-3 + (-2) + 1 + 2 = -2 < 0`，存在负环。

!!! danger "负环的后果"
    若源点到某点经过负环，则该点"最短路"为 `-∞`，问题无解。算法必须能**检测并报告**负环。

---

## 第二节 Bellman-Ford 算法

### 2.1 核心思想

**理论依据**：在不含负环的图中，从源点 s 到任意点 v 的最短路**最多经过 n-1 条边**（n 个点的简单路径最多 n-1 条边）。因此只需进行 **n-1 轮松弛**，每轮遍历所有边。

### 2.2 松弛操作

对边 `(u, v, w)`，松弛操作为：

```
if (dist[u] + w < dist[v])
    dist[v] = dist[u] + w;
```

**松弛的几何含义**：经过 u 中转，能否让 v 更近？

### 2.3 算法流程

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | 初始化 `dist[s]=0`，其余 `dist=+∞` | 源点距离为 0 |
| 2 | 循环 `n-1` 次 | 最多 n-1 条边 |
| 3 | 每轮遍历**所有边** `(u,v,w)`，尝试松弛 | 若 dist[u]+w<dist[v] 则更新 |
| 4 | 第 n 轮再扫一遍 | 若仍有松弛，则存在负环 |

### 2.4 代码实现

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge {
    int u, v, w;
};

int n, m, s;
vector<Edge> edges;
int dist[100005];

bool bellman_ford(int s) {
    memset(dist, 0x3f, sizeof(dist));
    dist[s] = 0;
    
    // n-1 轮松弛
    for (int i = 1; i <= n - 1; i++) {
        bool updated = false;
        for (auto &e : edges) {
            if (dist[e.u] != INF && dist[e.u] + e.w < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.w;
                updated = true;
            }
        }
        // 提前终止优化：本轮无更新则结束
        if (!updated) break;
    }
    
    // 第 n 轮检测负环
    for (auto &e : edges) {
        if (dist[e.u] != INF && dist[e.u] + e.w < dist[e.v]) {
            return false; // 存在负环
        }
    }
    return true; // 无负环
}

int main() {
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        edges.push_back({u, v, w});
    }
    // 输入示例：
    // 4 4 1
    // 1 2 2
    // 2 3 -3
    // 1 3 5
    // 3 4 1
    // 期望输出：0 2 -1 0
    
    if (bellman_ford(s)) {
        for (int i = 1; i <= n; i++)
            cout << dist[i] << " ";
    } else {
        cout << "存在负权环" << endl;
    }
    return 0;
}
```

!!! tip "提前终止优化"
    若某一轮没有任何距离被更新，说明已经收敛，可以提前结束。最坏情况仍需 n-1 轮，但实际运行往往快很多。

### 2.5 复杂度分析

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间 | O(nm) | n-1 轮，每轮 m 条边 |
| 空间 | O(n+m) | 存点距与边表 |

### 2.6 为什么 n-1 轮足够？

考虑源点 s 到点 v 的最短路 `s = p0 → p1 → ... → pk = v`（k ≤ n-1）。

- 第 1 轮松弛后，`dist[p1]` 一定被正确更新（因为 `dist[s]=0` 已确定）
- 第 2 轮松弛后，`dist[p2]` 一定被正确更新（因为 `dist[p1]` 已确定）
- ……
- 第 k 轮松弛后，`dist[v]` 一定被正确更新

因此 n-1 轮足以传播到所有点。

!!! info "松弛的传播性"
    Bellman-Ford 的精髓在于"逐层传播"：每一轮让最短路多走一步，类似于 BFS 的"层序扩展"，但允许边权为负。

---

## 第三节 SPFA：队列优化的 Bellman-Ford

### 3.1 优化动机

Bellman-Ford 的低效之处在于：每轮盲目扫描**所有边**，即使大部分点的距离早已确定。

**SPFA (Shortest Path Faster Algorithm)** 的观察：只有当 `dist[u]` 被更新时，u 的出边 `(u,v,w)` 才可能让 v 更新。因此只需把"刚被更新"的点入队，让它的邻居重新尝试松弛。

### 3.2 算法流程

```
1. dist[s] = 0，s 入队，标记 in_queue[s] = true
2. 队列非空时：
   a. 取出队首 u，取消标记 in_queue[u] = false
   b. 对 u 的每条出边 (u, v, w)：
      若 dist[u] + w < dist[v]：
        - dist[v] = dist[u] + w
        - 若 v 不在队中，v 入队，in_queue[v] = true
3. 队列空时结束
```

### 3.3 ASCII 流程示意

```
初始: dist[s]=0, 队列: [s]
     s ----2----> A
     |            |
     5            -1
     v            v
     B ----3----> C

第1轮: 弹出 s, 更新 A=2, B=5, 队列: [A, B]
第2轮: 弹出 A, 更新 C=2+(-1)=1, 队列: [B, C]
第3轮: 弹出 B, A? 5+3=8 > 2 不更新; B->C? 5+3=8 > 1 不更新
第4轮: 弹出 C, 无出边更新
队空, 结束. dist = {s:0, A:2, B:5, C:1}
```

### 3.4 代码实现

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge {
    int to, w;
};

int n, m, s;
vector<Edge> graph[100005];
int dist[100005];
bool in_queue[100005];

void spfa(int s) {
    memset(dist, 0x3f, sizeof(dist));
    memset(in_queue, false, sizeof(in_queue));
    dist[s] = 0;
    
    queue<int> q;
    q.push(s);
    in_queue[s] = true;
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        in_queue[u] = false;
        
        for (auto &e : graph[u]) {
            int v = e.to, w = e.w;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (!in_queue[v]) {
                    q.push(v);
                    in_queue[v] = true;
                }
            }
        }
    }
}

int main() {
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        graph[u].push_back({v, w});
    }
    // 输入示例：
    // 4 4 1
    // 1 2 2
    // 2 3 -3
    // 1 3 5
    // 3 4 1
    // 期望输出：0 2 -1 0
    
    spfa(s);
    for (int i = 1; i <= n; i++)
        cout << dist[i] << " ";
    return 0;
}
```

### 3.5 复杂度

| 维度 | 复杂度 | 说明 |
|------|--------|------|
| 时间（期望） | O(km)，k≈2 | 实际很快 |
| 时间（最坏） | O(nm) | 被特殊数据卡到退化 |
| 空间 | O(n+m) | 邻接表 |

!!! warning "SPFA 的名声"
    段凡丁 1994 年提出 SPFA 时声称复杂度为 O(km)，但后续研究发现最坏情况仍是 O(nm)。在 NOI/省选场景中，出题人常构造数据让 SPFA 退化。因此有名言：**"关于 SPFA，它死了。"**

---

## 第四节 SPFA 判负环

### 4.1 判定原理

若图中无负环，源点到任意点的最短路最多 n-1 条边。若某点**入队次数超过 n-1**，说明路径上出现重复点，即存在负环。

或者等价地：维护 `cnt[v]` 表示当前 `dist[v]` 对应路径的边数，若 `cnt[v] >= n`，则存在负环。

### 4.2 代码实现

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge { int to, w; };

int n, m;
vector<Edge> graph[2005];
int dist[2005], cnt[2005];
bool in_queue[2005];

// 返回 true 表示存在负环
bool spfa_negative_cycle() {
    memset(dist, 0x3f, sizeof(dist));
    memset(cnt, 0, sizeof(cnt));
    memset(in_queue, false, sizeof(in_queue));
    
    queue<int> q;
    // 注意：判断全图负环时，所有点都入队
    for (int i = 1; i <= n; i++) {
        dist[i] = 0;
        q.push(i);
        in_queue[i] = true;
    }
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        in_queue[u] = false;
        
        for (auto &e : graph[u]) {
            int v = e.to, w = e.w;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) return true; // 存在负环
                if (!in_queue[v]) {
                    q.push(v);
                    in_queue[v] = true;
                }
            }
        }
    }
    return false;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        graph[u].push_back({v, w});
    }
    // 输入示例：
    // 3 3
    // 1 2 -1
    // 2 3 -2
    // 3 1 -3
    // 期望输出：YES（存在负环）
    
    cout << (spfa_negative_cycle() ? "YES" : "NO") << endl;
    return 0;
}
```

!!! tip "判断全图负环 vs 源点可达负环"
    - 判断**从源点 s 出发是否经过负环**：只把 s 入队即可
    - 判断**全图是否存在负环**（无论从哪出发）：把所有点都入队

### 4.3 判负环的常见技巧

| 技巧 | 说明 |
|------|------|
| 所有源点入队 | 避免负环与源点不连通导致漏判 |
| `cnt[v] >= n` 触发 | 路径边数 ≥ n 必含环，且能松弛必为负环 |
| 入队次数上限 | 也可用 `入队次数 > n-1` 判断，但常数略大 |
| 随机化/分桶 | 极端数据下加速判负环 |

---

## 第五节 常见误区与陷阱

### 5.1 误区：SPFA 一定比 Bellman-Ford 快

理论上 SPFA 期望 O(km)，确实优于 Bellman-Ford 的 O(nm)。但**最坏情况两者相同**，出题人可以构造数据让 SPFA 退化。常见卡 SPFA 的图：

- **菊花图**：一个中心点连若干叶子，叶子反复入队
- **网格图**：均匀分布导致大量重复松弛
- **链式图+负权边**：迫使每一轮只前进一格

### 5.2 误区：负环判断用 dist 比较

错误写法：

```cpp
// ❌ 错误
if (dist[v] < -1e9) return true; // 存在负环
```

这种"距离足够小"的启发式**不可靠**——负权边密集的非负环图也可能让 dist 很小。必须用 `cnt[v] >= n` 或入队次数判断。

### 5.3 误区：Bellman-Ford 第 n 轮更新即负环

部分同学以为"第 n 轮任何更新都意味着负环"。准确说法是：**第 n 轮仍有松弛操作**，说明存在最短路边数 ≥ n 的路径，即含负环。

### 5.4 误区：SPFA 判负环只入队源点

若负环与源点 s 不连通，只入队 s 会漏判。判**全图负环**需把所有点入队；判**源点可达负环**则只入队源点。

### 5.5 误区：负权边一定有负环

负权边 ≠ 负权环。单条负权边只是让距离变小，但若所有环的权值和都 ≥ 0，则不存在负环。Bellman-Ford / SPFA 完全可以处理"有负权边但无负环"的图。

---

## 第六节 三种最短路算法对比

| 特性 | Dijkstra | Bellman-Ford | SPFA |
|------|----------|--------------|------|
| **适用图** | 非负权图 | 一般图（含负权） | 一般图（含负权） |
| **能否处理负权边** | ❌ 不能 | ✅ 能 | ✅ 能 |
| **能否检测负环** | ❌ 不能 | ✅ 能（第 n 轮） | ✅ 能（cnt≥n） |
| **时间复杂度** | O((n+m)log n) 堆优化 | O(nm) | 期望 O(km)，最坏 O(nm) |
| **空间复杂度** | O(n+m) | O(n+m) | O(n+m) |
| **实现难度** | 中等 | 简单 | 简单 |
| **稳定性** | 稳定 | 稳定 | 易被卡 |
| **常用场景** | 非负权最短路 | 理论分析、判负环 | 含负权的实际题目 |
| **数据结构** | 优先队列 | 边表 | 队列 + 邻接表 |
| **是否贪心** | ✅ 贪心 | ❌ 动态规划式松弛 | ❌ 队列驱动松弛 |

!!! tip "选择建议"
    - 题目明确**非负权** → 用 Dijkstra（堆优化）
    - 题目有**负权边但无负环** → SPFA（注意可能被卡，备选 Bellman-Ford）
    - 题目需要**判负环** → SPFA（小数据）或 Bellman-Ford（稳定）
    - 数据范围小且需稳定 → Bellman-Ford

---

## 第七节 典型例题

### 7.1 [P3385] 负环模板

**题意**：给定有向图，判断是否存在负环。

**思路**：SPFA 判负环模板题。把所有点入队，维护 `cnt[v]`，若 `cnt[v] >= n` 则存在负环。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge { int to, w; };

int T, n, m;
vector<Edge> graph[2005];
int dist[2005], cnt[2005];
bool in_queue[2005];

bool spfa() {
    memset(dist, 0x3f, sizeof(dist));
    memset(cnt, 0, sizeof(cnt));
    memset(in_queue, false, sizeof(in_queue));
    
    queue<int> q;
    // 判全图负环：所有点入队，避免负环与 1 不连通时漏判
    for (int i = 1; i <= n; i++) {
        dist[i] = 0;
        q.push(i);
        in_queue[i] = true;
    }
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        in_queue[u] = false;
        for (auto &e : graph[u]) {
            int v = e.to, w = e.w;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) return true;
                if (!in_queue[v]) {
                    q.push(v);
                    in_queue[v] = true;
                }
            }
        }
    }
    return false;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> T;
    while (T--) {
        cin >> n >> m;
        for (int i = 1; i <= n; i++) graph[i].clear();
        for (int i = 0; i < m; i++) {
            int u, v, w;
            cin >> u >> v >> w;
            graph[u].push_back({v, w});
            if (w >= 0) graph[v].push_back({u, w}); // 双向边只在非负时加
        }
        cout << (spfa() ? "YES" : "NO") << "\n";
    }
    return 0;
}
```

!!! warning "P3385 的细节"
    注意题目中 **w ≥ 0 的边为双向边，w < 0 的边为单向边**。这是常见坑点。

### 7.2 [P2136] 拉近距离

**题意**：n 个点 m 条有向边，每条边有长度。求从 1 到 n 的最短距离；若存在负环或不可达输出相应信息。

**思路**：标准 SPFA 单源最短路。注意处理负环与不可达两种特殊情况。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge { int to, w; };

int n, m;
vector<Edge> graph[5005];
int dist[5005], cnt[5005];
bool in_queue[5005];

int spfa() {
    memset(dist, 0x3f, sizeof(dist));
    memset(cnt, 0, sizeof(cnt));
    memset(in_queue, false, sizeof(in_queue));
    
    dist[1] = 0;
    queue<int> q;
    q.push(1);
    in_queue[1] = true;
    
    while (!q.empty()) {
        int u = q.front(); q.pop();
        in_queue[u] = false;
        for (auto &e : graph[u]) {
            int v = e.to, w = e.w;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) return -1; // 负环
                if (!in_queue[v]) {
                    q.push(v);
                    in_queue[v] = true;
                }
            }
        }
    }
    return dist[n] == INF ? -2 : dist[n]; // -2 表示不可达
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        graph[u].push_back({v, w});
    }
    int ans = spfa();
    if (ans == -1) cout << "Forever love" << endl;
    else if (ans == -2) cout << "No answer" << endl;
    else cout << ans << endl;
    return 0;
}
```

### 7.3 [P1819] 道路重建

**题意**：给定图与若干"必经边"，求从起点到终点的最短路径，使其经过所有必经边。

**思路**：由于必经边数量很少，可以**全排列必经边的顺序**，对每种顺序用 SPFA 求相邻段最短路，取最小值。注意负环检测与状态处理。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;
struct Edge { int to, w; };

int n, m, k;
vector<Edge> graph[10005];
int must[10];

// SPFA 求 from 到 to 的最短路
int spfa(int from, int to) {
    if (from == to) return 0;
    vector<int> dist(n + 1, INF);
    vector<bool> in_queue(n + 1, false);
    queue<int> q;
    dist[from] = 0;
    q.push(from);
    in_queue[from] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        in_queue[u] = false;
        for (auto &e : graph[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                if (!in_queue[e.to]) {
                    q.push(e.to);
                    in_queue[e.to] = true;
                }
            }
        }
    }
    return dist[to];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m >> k;
    for (int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        graph[u].push_back({v, w});
        graph[v].push_back({u, w});
    }
    for (int i = 0; i < k; i++) cin >> must[i];
    
    // 全排列必经点，求最小总距离
    sort(must, must + k);
    int ans = INF;
    do {
        int total = spfa(1, must[0]);
        bool ok = (total < INF);
        for (int i = 1; i < k && ok; i++) {
            int d = spfa(must[i - 1], must[i]);
            if (d >= INF) ok = false;
            else total += d;
        }
        if (ok) {
            int d = spfa(must[k - 1], n);
            if (d < INF) ans = min(ans, total + d);
        }
    } while (next_permutation(must, must + k));
    
    cout << (ans == INF ? -1 : ans) << endl;
    return 0;
}
```

!!! info "状态压缩思路"
    当必经边/必经点数 ≤ 20 时，可改用**状压 DP**：`dp[mask][v]` 表示已访问必经点集合为 mask、当前在 v 的最短距离。复杂度 O(2^k · k · (n+m))。

---

## 第八节 进阶：SPFA 的优化技巧

当数据较"硬"导致 SPFA 退化时，可尝试以下优化（但仍不能保证不被卡）：

### 8.1 SLF (Small Label First)

队首入队时，若新点距离小于队首距离，插入队首；否则插入队尾。

```cpp
// 使用 deque 替代 queue
deque<int> q;
if (!q.empty() && dist[v] < dist[q.front()])
    q.push_front(v);
else
    q.push_back(v);
```

### 8.2 LLL (Large Label Last)

若队首距离大于队列平均距离，移到队尾。

### 8.3 容错退路

!!! danger "终极建议"
    题目保证非负权时，**永远优先 Dijkstra**。SPFA 只在必须处理负权时使用，且要警惕被卡。

---

### 知识衔接

本章是**最短路算法系列**的第二章，与前后章节的联系如下：

- **前置知识**：第30章Dijkstra算法（最短路基础框架）、第29章拓扑排序（DAG判环思想）、第26章图的存储与遍历（松弛操作的图基础）
- **本章定位**：解决含负权图的单源最路与负环判定，是Dijkstra无法处理负权时的关键补充
- **后续发展**：为第32章Floyd（全源最短路）提供单源基础，SPFA在差分约束系统中是核心求解工具

## 📖 本章总结

```
第31章 最短路II：Bellman-Ford & SPFA
├── Bellman-Ford 算法
│   ├── 核心：n-1 轮全边松弛，O(nm)
│   ├── 判负环：第 n 轮仍能松弛 → 存在负环
│   ├── 提前终止：某轮无更新可 break
│   └── 适用：一般图（含负权边），稳定但慢
├── SPFA 算法
│   ├── 核心：队列优化，只处理"被更新"的点
│   ├── 复杂度：期望 O(km)，最坏 O(nm)
│   ├── 判负环：cnt[v] >= n 或入队次数 > n-1
│   └── 风险：易被特殊数据卡到退化
├── 判负环策略
│   ├── 判全图负环：所有点入队
│   └── 判源点可达负环：只入队源点
├── 常见误区
│   ├── 用 Dijkstra 处理负权图 → 改用 SPFA/Bellman-Ford
│   ├── SPFA 判负环只入队源点 → 判全图负环需所有点入队
│   ├── 用 dist 大小判负环 → 用 cnt[v] >= n 判定
│   ├── 忽略 INF 相加溢出 → 松弛前判断 dist[u] != INF
│   └── 优先用 SPFA 而非 Dijkstra → 非负权必用 Dijkstra
└── 配套习题
    ├── P3385 负环模板（注意 w≥0 双向边）
    ├── P2136 拉近距离（负环/不可达分别处理）
    └── P1819 道路重建（全排列必经边 + SPFA）
```

### 算法选择决策树

```
图中是否有负权边？
├── 否 → Dijkstra (堆优化) O((n+m)log n)
└── 是 → 是否需要判负环？
        ├── 是 → SPFA 判负环（小数据）/ Bellman-Ford（稳定）
        └── 否 → SPFA 求最短路（警惕被卡）
                  └── 若被卡 → 退回 Bellman-Ford
```

### 易错点清单

| 编号 | 易错点 | 正确做法 |
|------|--------|----------|
| 1 | 用 Dijkstra 处理负权图 | 改用 SPFA 或 Bellman-Ford |
| 2 | SPFA 判负环只入队源点 | 判全图负环需所有点入队 |
| 3 | 用 dist 大小判负环 | 用 `cnt[v] >= n` 判定 |
| 4 | Bellman-Ford 不做提前终止 | 加 `updated` 标志优化 |
| 5 | 忽略 INF 相加溢出 | 松弛前判断 `dist[u] != INF` |
| 6 | 双向边误加（P3385） | 注意 w<0 时只加单向边 |
| 7 | 优先用 SPFA 而非 Dijkstra | 非负权必用 Dijkstra |

### 自测清单

!!! tip "能做到以下几点，才算真正掌握本章"
    - [ ] 解释 Dijkstra 在负权图失效的原因，能画出反例
    - [ ] 默写 Bellman-Ford 的 n-1 轮松弛 + 第 n 轮判负环
    - [ ] 默写 SPFA 的队列实现，说明 `in_queue` 的作用
    - [ ] 区分"判全图负环"与"判源点可达负环"的入队策略
    - [ ] 说出 SPFA 最坏退化 O(nm) 的原因
    - [ ] 根据题意在三种算法中正确选择

### 下章预告

下一章将学习**全源最短路 Floyd 算法**，它能在 O(n³) 时间内求出图中**所有点对**之间的最短路，并能基于"传递闭包"判断连通性与可达性。Floyd 的动态规划思想也将为后续图论学习奠定基础。

---

> **练习建议**：完成 P3385、P2136、P1819 三题，重点体会负环判定的细节与 SPFA 的实现。在 OJ 上尝试用 SPFA 提交非负权最短路题，再换 Dijkstra 对比运行时间，直观感受"SPFA 已死"的含义。