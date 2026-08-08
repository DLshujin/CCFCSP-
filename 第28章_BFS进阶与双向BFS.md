# 第28章 BFS进阶与双向BFS

!!! info "本章内容"
    广度优先搜索（BFS）是信息学竞赛中最基础也最重要的算法之一。在第26章图遍历的基础上，本章深入探讨 BFS 在最短路径、状态空间搜索中的应用，并引入强大的双向 BFS 技巧，应对状态爆炸问题。你将学习：

    - BFS 求无权图最短路的原理与正确性证明
    - 状态空间搜索：把"问题"变成"图"
    - 双向 BFS 的核心思想与实现要点
    - 八数码问题的状态压缩与双向 BFS 解法
    - visited 标记时机的经典误区

!!! warning "本章定位"
    BFS 是 CSP-S 图论与搜索题的"万金油"——无权图最短路、最小步数、层序遍历都离不开它。双向 BFS 则是应对状态爆炸的利器，能把复杂度从 $O(b^d)$ 降到 $O(b^{d/2})$。本章要求：**BFS 模板必须默写**，**双向 BFS 的按层扩展与相遇判断必须理解**，**八数码代码必须独立写一遍**。

---

## 📌 学习目标

学完本章后，你应当能够：

1. **解释**为什么 BFS 首次访问即最短路径，并能在无权图中用 BFS 求最短路
2. **默写**BFS 标准模板（方向数组 + 队列 + 入队即标记），说明每个细节的作用
3. **将实际问题**抽象为状态空间搜索（状态表示 + 转移 + 判重），独立解决马的遍历等题目
4. **解释**双向 BFS 的原理与复杂度优势，能分析为什么是 $O(b^{d/2})$ 而非 $O(b^d)$
5. **默写**双向 BFS 框架代码（交替扩展较小一侧 + 按层扩展 + 相遇判断）
6. **独立写出**八数码问题的双向 BFS 解法，掌握状态压缩技巧
7. **避开**visited 标记时机、层扩展遗漏、相遇判断遗漏等常见误区

---

## 💡 生活类比

### 类比一：水波纹扩散

想象往平静的湖面投一颗石子，**水波纹一圈圈向外扩散**。最先触达某片落叶的那一圈波纹，必然经过最短的"水路"。BFS 正是这种"水波纹扩散"——按距离层次依次扩展，第 $k$ 层的节点距离源点恰为 $k$。

```
        源点 s (第0层)
       /    \
      A      B      (第1层, 距离=1)
     / \    / \
    C   D  E   F    (第2层, 距离=2)
         |
         G         (第3层, 距离=3)
```

### 类比二：商场找人

你和朋友在偌大的商场走散，要找彼此。两人都站在原地不动，你一个人满商场找，要走很多路（单向 BFS）。但若两人**同时朝对方方向走**、还互相打电话确认方位，相遇所需走的总路程大幅减少（双向 BFS）。

!!! tip "一句话记忆"
    BFS = 水波纹扩散，一层层往外走，先碰到的一定最近。双向 BFS = 两人同时出发，中间相遇，各走一半路。

---

## 第一节 BFS 求最短路径（无权图）

### 1.1 核心结论

**在无权图（或所有边权相等的图）中，BFS 首次访问某节点时所走的路径，就是从源点到该节点的最短路径。**

!!! info "正确性直觉"
    设源点为 $s$。BFS 按"先进先出"顺序处理队列，先入队的节点距离不会比后入队的节点大。

    - 初始：$s$ 入队，`dist[s] = 0`，正确。
    - 归纳：假设当前所有已确定 `dist` 的节点都取到了最短距离，那么从这些节点出发的边所指向的邻居，其 `dist` = 当前节点 `dist + 1`，也是最小的（因为不可能有更短的路径，否则该邻居早已在前一层被访问）。

    因此 **首次访问 = 最短路径**，这是 BFS 区别于 DFS 的关键性质。

!!! tip "与 Dijkstra 的关系"
    当所有边权相等时，Dijkstra 退化为 BFS。所以无权图最短路用 BFS 即可，无需优先队列，复杂度 $O(n+m)$。

### 1.2 层次遍历示意

以一个 5×5 迷宫为例，`S` 为起点，`T` 为终点，`#` 为障碍：

```
S . . # .
. # . # .
. # . . .
. # # # .
. . . . T
```

BFS 逐层扩展，`dist` 值标注如下：

```
0 1 2 # 8
1 # 3 # 7
2 # 4 5 6
3 # # # 7
4 5 6 7 8
            ↑ T 在第8层, 最短步数=8
```

队列演变（按层次）：
- 第0层：`[S]`，dist=0
- 第1层：扩展 S 得 `[(1,2),(2,1)]`
- 第2层：扩展第1层得 `[(1,3),(3,1)]`
- ……
- 第8层：到达 T，返回 dist=8

---

## 第二节 BFS 标准模板

### 2.1 迷宫最短路模板

```cpp
#include <bits/stdc++.h>
using namespace std;

const int dx[] = {-1, 1, 0, 0};
const int dy[] = {0, 0, -1, 1};

int n, m;
char mp[105][105];
int dist[105][105];
bool vis[105][105];

int bfs(int sx, int sy, int ex, int ey) {
    queue<pair<int,int>> q;
    q.push({sx, sy});
    vis[sx][sy] = true;          // 入队时立即标记
    dist[sx][sy] = 0;
    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        if (x == ex && y == ey) return dist[x][y];
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx < 1 || nx > n || ny < 1 || ny > m) continue;
            if (mp[nx][ny] == '#' || vis[nx][ny]) continue;
            vis[nx][ny] = true;           // 关键：入队即标记
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    return -1;  // 不可达
}
// 预期：输入迷宫起点终点，输出最短步数，不可达输出 -1
```

### 2.2 模板要点

| 要点 | 说明 |
|------|------|
| **方向数组** | `dx[], dy[]` 让代码简洁，避免写 4 个 if |
| **边界检查顺序** | 先判越界，再判障碍，最后判访问 |
| **距离传递** | `dist[nx][ny] = dist[x][y] + 1` |
| **入队即标记** | `vis[nx][ny] = true` 必须在 `push` 之前，详见第六节 |

---

## 第三节 BFS 与状态图搜索

### 3.1 把"问题"变成"图"

很多题目表面不是图论题，但可以**将每个状态抽象为图中的一个节点**，状态之间的转移抽象为边。这就是"状态空间搜索"。

| 题目类型 | 状态（节点） | 转移（边） |
|---------|------------|----------|
| 迷宫最短路 | 当前坐标 $(x,y)$ | 上下左右移动 |
| 马的遍历 | 棋盘上的位置 | 马走"日"的 8 个方向 |
| 八数码 | 棋盘排列（9! 种） | 0 与相邻格交换 |
| 倒水问题 | $(水量_A, 水量_B)$ | 倒满/倒空/互倒 |
| 字词接龙 | 当前单词 | 改一个字母得到新词 |

### 3.2 状态搜索三要素

| 步骤 | 操作 | 数据结构 |
|-----|------|---------|
| 1. 状态表示 | 用整数、字符串、tuple 等编码一个局面 | — |
| 2. 状态判重 | 记录已访问状态，避免重复搜索 | `bool[]` / `unordered_set` |
| 3. 状态扩展 | 枚举所有合法转移，生成新状态 | 循环 |

!!! warning "状态空间爆炸"
    状态数往往随规模急剧增长。例如八数码有 $9!/2 = 181440$ 个可达状态，$4 \times 4$ 数字华容道则多达 $10^{13}$。状态过多时必须用双向 BFS 或 A* 等优化。

---

## 第四节 双向 BFS

### 4.1 为什么要双向

普通 BFS 从起点单向扩展，搜索树随深度呈指数增长。如果起点和终点都已知，可以**同时从两端搜索**，当两股搜索在中间"相遇"时即得答案。

### 4.2 复杂度对比

设分支因子为 $b$，最短路径长度为 $d$：

| 方法 | 扩展节点数（量级） | 说明 |
|-----|------------------|------|
| 单向 BFS | $O(b^d)$ | 搜索树深度 $d$ |
| **双向 BFS** | $O(b^{d/2})$ | 两棵深度 $d/2$ 的树 |

当 $d$ 较大时，$b^{d/2}$ 远小于 $b^d$。例如 $b=4, d=20$：单向约 $10^{12}$，双向约 $10^6$，**加速百万倍**。

### 4.3 相遇示意图

```
单向 BFS:
s * * * * * * * * * * * * * * * * * * * t   (扩展20层)

双向 BFS:
s * * * * * * * * *               * * * * * * * * * t
              <--- 相遇 --->        (各扩展10层)
```

---

## 第五节 双向 BFS 实现要点

### 5.1 核心思想

维护两个队列 `qA`（从起点出发）和 `qB`（从终点出发），交替扩展。每扩展一层，检查新节点是否出现在另一侧的已访问集合中——若出现，说明"相遇"，最短路 = 两端距离之和。

### 5.2 交替扩展策略

为保证效率，**每次扩展节点数较少的那一侧**。这样两侧搜索树大小均衡，相遇点接近中点。

!!! tip "为什么交替而非并行"
    严格一层层交替扩展，能保证找到的是最短路。若两侧扩展深度不均（比如一边已扩 5 层，另一边才 1 层），相遇时可能错过最短路径。每次取较小一侧扩展，天然保持平衡。

### 5.3 相遇判断

- 用两个 `visited` 集合（或 map）记录各自已访问状态及其距离
- 扩展节点 $v$ 时，若 $v$ 在对侧 `visited` 中，则：`答案 = distA[v] + distB[v]`

### 5.4 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

// 双向 BFS 框架（伪状态接口）
unordered_map<State,int> visA, visB;  // 状态 -> 距离

int bidirectional_bfs(State S, State T) {
    if (S == T) return 0;
    queue<State> qA, qB;
    qA.push(S); visA[S] = 0;
    qB.push(T); visB[T] = 0;

    while (!qA.empty() && !qB.empty()) {
        // 选择较小的一侧扩展一层
        if (qA.size() <= qB.size()) {
            int res = expand(qA, visA, visB);
            if (res != -1) return res;
        } else {
            int res = expand(qB, visB, visA);
            if (res != -1) return res;
        }
    }
    return -1;
}

// expand: 从 from 队列扩展一层，检查是否命中 to 的 visited
int expand(queue<State>& from, unordered_map<State,int>& visFrom,
           unordered_map<State,int>& visTo) {
    int sz = from.size();
    for (int i = 0; i < sz; i++) {        // 只扩展当前层
        State u = from.front(); from.pop();
        int d = visFrom[u];
        for (State v : nextStates(u)) {
            if (visFrom.count(v)) continue;  // 已在本侧访问
            if (visTo.count(v))              // 命中对侧
                return d + 1 + visTo[v];
            visFrom[v] = d + 1;
            from.push(v);
        }
    }
    return -1;
}
// 预期：输入起点 S 和终点 T，输出最短步数，不可达返回 -1
```

!!! warning "层扩展 vs 逐个扩展"
    必须按"层"扩展（一次处理当前队列里全部 `sz` 个元素），否则无法保证首次相遇即最短。逐个扩展可能让相遇点偏离最优。

### 5.5 逐步追踪示例

以一个简化的一维状态图为例，状态为整数，转移为 $\pm 1$。求从 0 到 10 的最少步数：

```
初始:
  qA = [0],   visA = {0:0}
  qB = [10],  visB = {10:0}

第1轮 (两侧都扩展1层):
  expand(qA): 0 -> 1,   visA = {0:0, 1:1},  qA=[1]
  expand(qB): 10 -> 9,  visB = {10:0, 9:1}, qB=[9]

第2轮:
  expand(qA): 1 -> 2,   visA = {0:0,1:1,2:2},  qA=[2]
  expand(qB): 9 -> 8,   visB = {10:0,9:1,8:2}, qB=[8]

  ...  (继续交替扩展)

第5轮:
  expand(qA): 4 -> 5,   visA = {...,5:5},  qA=[5]
  expand(qB): 6 -> 5,   5 已在 visA 中! 相遇!
    答案 = visA[5] + visB[5] = 5 + 5 = 10 ✓
```

两端各扩展 5 层即可相遇，相比单向 BFS 扩展 10 层，节点数大幅减少。

---

## 第六节 常见误区

### 误区一：visited 标记时机错误（最经典）

| 写法 | 何时标记 | 复杂度 | 正确性 |
|-----|---------|-------|-------|
| ✅ 正确 | **入队时**标记 | $O(n+m)$ | 正确 |
| ❌ 错误 | **出队时**标记 | 可能 $O(n^2)$ 退化 | 正确但低效 |

```cpp
// ❌ 错误：出队时才标记
while (!q.empty()) {
    auto [x, y] = q.front(); q.pop();
    if (vis[x][y]) continue;     // 出队时才判重
    vis[x][y] = true;            // 出队时才标记
    for (...) {
        q.push({nx, ny});        // 同一节点可能被多次入队！
    }
}

// ✅ 正确：入队时立即标记
while (!q.empty()) {
    auto [x, y] = q.front(); q.pop();
    for (...) {
        if (vis[nx][ny]) continue;
        vis[nx][ny] = true;     // 入队时立即标记
        q.push({nx, ny});
    }
}
```

**问题**：出队时才标记，同一节点会被多次入队（每次邻居扩展时都入队一次），队列膨胀到 $O(E)$，浪费时间。

!!! tip "记忆口诀"
    "先标记，再入队"——一个节点只要进了队列，就立刻"上户口"，绝不让它第二次进队。

### 误区二：双向 BFS 不按层扩展

```cpp
// ❌ 错误：逐个扩展，不按层
int sz = 1;  // 或没有 sz 的概念
for (...) {
    State u = from.front(); from.pop();
    // 直接扩展，不控制层次
}

// ✅ 正确：用 sz 控制只扩展一层
int sz = from.size();
for (int i = 0; i < sz; i++) {
    State u = from.front(); from.pop();
    // 扩展当前层所有节点
}
```

**后果**：不按层扩展会导致相遇点偏离最优，答案不是最短路径。

### 误区三：相遇判断遗漏

```cpp
// ❌ 错误：扩展新状态时忘了检查对侧 visited
for (State v : nextStates(u)) {
    if (visFrom.count(v)) continue;
    visFrom[v] = d + 1;
    from.push(v);
    // 忘了检查 visTo.count(v)！
}

// ✅ 正确：扩展时立即检查对侧
for (State v : nextStates(u)) {
    if (visFrom.count(v)) continue;
    if (visTo.count(v))              // 命中对侧！
        return d + 1 + visTo[v];
    visFrom[v] = d + 1;
    from.push(v);
}
```

### 误区四：方向数组写错

马的 8 个方向容易写错，建议画图验证：

```cpp
// 马走"日"的 8 个方向
const int dx[] = {-2, -1, 1, 2, 2, 1, -1, -2};
const int dy[] = {1, 2, 2, 1, -1, -2, -2, -1};
```

### 误区五：边界 `>` 写成 `>=`

```cpp
// ❌ 错误：nx >= n 会排除合法的 nx==n
if (nx < 1 || nx >= n || ny < 1 || ny >= m) continue;

// ✅ 正确：nx > n 才越界，保留 nx==n
if (nx < 1 || nx > n || ny < 1 || ny > m) continue;
// 或等价写法：if (nx >= 1 && nx <= n && ny >= 1 && ny <= m) { ... }
```

### 误区六：八数码状态拼整数丢失前导 0

```cpp
// ❌ 错误：用整数拼状态，前导 0 丢失
int state = 12380465;  // 实际是 012380465，前导 0 丢了

// ✅ 正确：用固定 9 位拼接，或用字符串/unordered_map
// 方法1：用字符串 "123804765"
// 方法2：确保整数按 9 位处理（从高位到低位逐位解析）
```

---

## 第七节 经典例题：八数码问题（P1379）

### 7.1 题意

在 $3 \times 3$ 棋盘上摆有 1~8 八个数字和一个空格（用 0 表示）。每次可将空格与上下左右相邻的数字交换。给定初始状态，求最少步数使棋盘变为：

```
1 2 3
8 0 4
7 6 5
```

即目标状态为 `123804765`（按行拼接）。

### 7.2 思路

- **状态压缩**：9 个数字拼成 9 位整数，用一个 `int` 即可表示一个状态
- **判重**：用 `unordered_map` 记录已访问状态及距离
- **双向 BFS**：从初态和目标态同时扩展，相遇即得答案

!!! tip "可达性判定（扩展知识）"
    八数码有解当且仅当初始状态逆序对数的奇偶性与目标状态相同。本题保证有解，可省略检查。

### 7.3 完整 AC 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int TARGET = 123804765;
// pos 的4个邻居(棋盘位置0..8)
const vector<vector<int>> adj = {
    {1, 3},       // 0
    {0, 2, 4},    // 1
    {1, 5},       // 2
    {0, 4, 6},    // 3
    {1, 3, 5, 7}, // 4
    {2, 4, 8},    // 5
    {3, 7},       // 6
    {4, 6, 8},    // 7
    {5, 7}        // 8
};

int find_zero(int state) {
    for (int i = 8; i >= 0; i--) {
        if (state % 10 == 0) return i;
        state /= 10;
    }
    return -1;
}

int swap_pos(int state, int p, int q) {
    int a[9];
    for (int i = 8; i >= 0; i--) { a[i] = state % 10; state /= 10; }
    swap(a[p], a[q]);
    int ret = 0;
    for (int i = 0; i < 9; i++) ret = ret * 10 + a[i];
    return ret;
}

unordered_map<int,int> visA, visB;

int expand(queue<int>& q, unordered_map<int,int>& visMe,
           unordered_map<int,int>& visOther) {
    int sz = q.size();
    for (int i = 0; i < sz; i++) {
        int u = q.front(); q.pop();
        int d = visMe[u];
        int z = find_zero(u);
        for (int npos : adj[z]) {
            int v = swap_pos(u, z, npos);
            if (visMe.count(v)) continue;
            if (visOther.count(v))
                return d + 1 + visOther[v];   // 相遇
            visMe[v] = d + 1;
            q.push(v);
        }
    }
    return -1;
}

int main() {
    int start; cin >> start;
    if (start == TARGET) { cout << 0; return 0; }

    queue<int> qA, qB;
    qA.push(start); visA[start] = 0;
    qB.push(TARGET); visB[TARGET] = 0;

    while (!qA.empty() && !qB.empty()) {
        int res;
        if (qA.size() <= qB.size())
            res = expand(qA, visA, visB);
        else
            res = expand(qB, visB, visA);
        if (res != -1) { cout << res; return 0; }
    }
    cout << -1;
    return 0;
}
// 预期：输入初始状态（如 283104765），输出最少步数
```

### 7.4 复杂度分析

- 状态总数：$9!/2 = 181440$（可达状态）
- 单向 BFS 最坏扩展全部状态
- 双向 BFS 从两端各扩展约 $b^{d/2}$ 个状态即可相遇（$b$ 为分支因子，$d$ 为深度），远少于单向 BFS 的 $b^d$，**显著加速**

---

## 第八节 📚 本节练习

### 8.1 [P1443] 马的遍历

**题意**：$n \times m$ 棋盘，马从 $(x,y)$ 出发，求到每个格子的最少步数，不可达输出 -1。

**思路**：标准 BFS。马的 8 个方向用方向数组表示，从起点出发层次遍历即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int dx[] = {-2,-1,1,2,2,1,-1,-2};
const int dy[] = {1,2,2,1,-1,-2,-2,-1};

int n, m, sx, sy;
int dist[405][405];
bool vis[405][405];

int main() {
    cin >> n >> m >> sx >> sy;
    memset(dist, -1, sizeof(dist));
    queue<pair<int,int>> q;
    q.push({sx, sy});
    vis[sx][sy] = true;
    dist[sx][sy] = 0;
    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        for (int i = 0; i < 8; i++) {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx < 1 || nx > n || ny < 1 || ny > m) continue;
            if (vis[nx][ny]) continue;
            vis[nx][ny] = true;
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++)
            cout << left << setw(5) << dist[i][j];
        cout << endl;
    }
    return 0;
}
// 预期：输入棋盘大小和起点，输出每个格子的最短步数矩阵
```

!!! tip "格式化输出"
    使用 `setw(5)` + `left` 对齐输出，避免末尾多余空格被判错。

### 8.2 [P1747] 好奇怪的游戏

**题意**：棋盘上两匹马分别从不同起点出发，求两匹马都到达 $(1,1)$ 的最少总步数。马可走"日"字或走"田"字（共 12 个方向）。

**思路**：分别对两匹马做 BFS 求 $(1,1)$ 的最短步数，相加即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
// 8个"日"字方向 + 4个"田"字方向 = 12方向
const int dx[] = {-2,-1,1,2,2,1,-1,-2,-2,-2,2,2};
const int dy[] = {1,2,2,1,-1,-2,-2,-1,-2,2,-2,2};

int dist[25][25];
bool vis[25][25];

int bfs(int sx, int sy) {
    memset(vis, 0, sizeof(vis));
    memset(dist, -1, sizeof(dist));
    queue<pair<int,int>> q;
    q.push({sx, sy});
    vis[sx][sy] = true;
    dist[sx][sy] = 0;
    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        if (x == 1 && y == 1) return dist[x][y];
        for (int i = 0; i < 12; i++) {
            int nx = x + dx[i], ny = y + dy[i];
            if (nx < 1 || ny < 1) continue;
            if (nx > 22 || ny > 22) continue;
            if (vis[nx][ny]) continue;
            vis[nx][ny] = true;
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    return -1;
}

int main() {
    int x1, y1, x2, y2;
    cin >> x1 >> y1 >> x2 >> y2;
    cout << bfs(x1, y1) + bfs(x2, y2);
    return 0;
}
// 预期：输入两匹马起点坐标，输出最少总步数
```

### 8.3 [P1135] 奇怪的电梯

**题意**：大楼 $n$ 层，电梯在第 $i$ 层只能上/下 $K_i$ 层（不能超出 $1 \sim n$）。求从 $A$ 层到 $B$ 层最少按键次数。

**思路**：状态图 BFS。每层看作一个节点，转移为 $i \pm K_i$。用 `dist` 数组判重并记录步数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 205;
int n, A, B, k[N];
int dist[N];

int bfs() {
    memset(dist, -1, sizeof(dist));
    queue<int> q;
    q.push(A); dist[A] = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        if (u == B) return dist[u];
        for (int d : {k[u], -k[u]}) {
            int v = u + d;
            if (v < 1 || v > n) continue;
            if (dist[v] != -1) continue;
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }
    return -1;
}

int main() {
    cin >> n >> A >> B;
    for (int i = 1; i <= n; i++) cin >> k[i];
    cout << bfs() << endl;
    return 0;
}
// 预期：输入 n, A, B 和各层 K 值，输出最少按键次数，不可达输出 -1
```

---

## 📖 本章总结

```
第28章 BFS进阶与双向BFS
├── BFS 求最短路
│   ├── 无权图中 BFS 首次访问 = 最短路径
│   ├── 与 Dijkstra 的关系：等权时 Dijkstra 退化为 BFS
│   └── 复杂度 O(n+m)
├── BFS 标准模板
│   ├── 方向数组 dx[], dy[]
│   ├── 队列 + visited 标记
│   ├── 入队即标记（关键！）
│   └── 边界检查顺序：越界 → 障碍 → 访问
├── 状态空间搜索
│   ├── 三要素：状态表示、状态判重、状态扩展
│   ├── 状态压缩技巧（整数、字符串、tuple）
│   └── 状态爆炸时需双向 BFS / A*
├── 双向 BFS
│   ├── 核心思想：起终点同时扩展，中间相遇
│   ├── 复杂度：O(b^d) → O(b^(d/2))
│   ├── 交替扩展较小一侧
│   ├── 按层扩展（sz 控制）
│   └── 相遇判断：visA[v] + visB[v]
├── 常见误区
│   ├── visited 标记时机：入队即标记，非出队
│   ├── 双向 BFS 不按层扩展 → 答案非最优
│   ├── 相遇判断遗漏 → 漏答或超时
│   ├── 方向数组写错 → 答案错误
│   └── 八数码状态压缩丢前导 0
└── 配套习题
    ├── P1443 马的遍历：BFS 模板
    ├── P1747 好奇怪的游戏：多方向 BFS
    ├── P1135 奇怪的电梯：状态图 BFS
    └── P1379 八数码：双向 BFS 经典题
```

### 自测清单（合上书自检）

!!! tip "能做到以下几点，才算真正掌握本章"
    - [ ] 解释为什么 BFS 首次访问即最短路径
    - [ ] 默写 BFS 标准模板（方向数组 + 队列 + 入队即标记）
    - [ ] 说出 visited 标记的正确时机（入队时），并解释为什么
    - [ ] 将"马的遍历""奇怪电梯"等实际问题抽象为状态空间 BFS
    - [ ] 解释双向 BFS 的复杂度优势 $O(b^{d/2})$ vs $O(b^d)$
    - [ ] 默写双向 BFS 框架（交替扩展 + 按层扩展 + 相遇判断）
    - [ ] 独立写出八数码的双向 BFS 解法
    - [ ] 说出双向 BFS 中按层扩展的重要性（不按层会导致答案非最优）

### 选择策略速查

| 场景 | 推荐方法 |
|-----|---------|
| 普通迷宫/网格最短路 | 单向 BFS |
| 已知起终点，状态空间大 | 双向 BFS |
| 状态数爆炸（如 15 数码） | 双向 BFS + A* |
| 边权不等 | Dijkstra / 0-1 BFS |

### 知识衔接

本章是**广度优先搜索的系统化进阶**，与前后章节的联系如下：

- **前置知识**：第27章DFS进阶与剪枝（搜索策略对比）、第19章队列（BFS的底层数据结构）、第26章图的存储与遍历（BFS基础框架）
- **本章定位**：将朴素BFS升级为双向BFS、分层BFS等高效策略，是无权图最短路径问题的核心解法
- **后续发展**：为第29章拓扑排序（Kahn算法基于BFS）提供基础，BFS思想在第30章Dijkstra分层扩展中也有体现

!!! info "下一章预告"
    下一章将学习**拓扑排序**——处理"有依赖关系"的排序问题。拓扑排序与 BFS 一脉相承（Kahn 算法本质就是 BFS），并在 DAG 上结合 DP 解决最长路、路径计数等经典问题，是 CSP-S 图论的另一块基石。