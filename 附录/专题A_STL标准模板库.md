# 专题A STL标准模板库：竞赛编码的瑞士军刀

> STL 是 C++ 标准库的精华，也是信息学竞赛选手的"瑞士军刀"。掌握 STL，你就能用几十行代码完成手写几百行才能实现的功能，把精力集中在算法本身。本专题位于第26章（图的存储与遍历）之后、第27章（DFS进阶）之前，作为后续算法学习的工具基石。

!!! info "本专题内容"
    本专题系统讲解 CSP-S 必考的 STL 组件：三大组件概述、序列容器（vector/string/pair）、容器适配器（stack/queue/priority_queue）、有序关联容器（set/multiset/map）、无序关联容器（unordered_set/unordered_map）、迭代器与范围 for 循环、常用算法（sort/binary_search/unique/next_permutation 等）、常见误区汇总与速查总表。

!!! warning "学习定位"
    学完第18~26章的数据结构后，你会发现 STL 已经把这些数据结构"打包"好了——`stack` 就是手写栈的成品版，`priority_queue` 就是堆的成品版。本专题的目标是让你**会用现成工具**，在后续算法章节中专注算法逻辑而非造轮子。建议学完本专题后再进入第27章 DFS 进阶。

---

## 📌 学习目标

学完本专题后，你应当能够：

1. **准确说出** STL 三大组件（容器、迭代器、算法）的职责分工，并能区分序列容器、适配器、关联容器的特点
2. **默写** vector、stack、queue、priority_queue、set、map 的核心操作与时间复杂度
3. **独立编写** 使用 `sort` + 自定义比较函数、`lower_bound`/`upper_bound` 二分查找、`unique` 去重的完整程序
4. **解释** `map[]` 会插入默认值、`priority_queue` 默认大根堆、迭代器失效等常见坑点，并给出正确写法
5. **根据题目特点**选择 set 与 unordered_set、map 与 unordered_map，并完成至少 5 道洛谷 STL 练习题

---

## 💡 生活类比

### 类比：一家"工具齐全"的厨房

把写程序比作做菜。**手写数据结构**就像从种菜、养鸡开始，自己造锅造刀——费力且容易做坏；**用 STL** 就像走进一家工具齐全的厨房：

- **vector** = 一口**能伸缩的炒锅**，菜多了自动变大，菜少了也不浪费
- **stack** = 一摞**盘子**，只能从最上面拿放（后进先出）
- **queue** = 排队取餐的**队伍**，先到的先打饭（先进先出）
- **priority_queue** = 医院叫号系统，**病情重的优先**（堆）
- **set** = 一本**按字母排序的通讯录**，查人快、不重名
- **map** = 一本**字典**，按词条（key）查释义（value），词条自动排序
- **unordered_set/map** = 一本**乱序但翻得飞快**的字典，用哈希表实现
- **sort** = 一个**专业分拣员**，O(n log n) 帮你把货排好
- **lower_bound** = 一个**精准定位员**，在有序货物里二分找位置

!!! tip "一句话记住"
    STL = **现成的工具箱**。竞赛不是比谁造工具快，而是比谁用工具解题快。先学会用，再学原理（手写实现）。

---

## 核心概念

### A.1 STL 概述

「STL Standard Template Library 标准模板库」是 C++ 标准库的核心部分，提供了一组通用的**模板化**数据结构与算法。它由三大组件构成：

| 组件 | 作用 | 类比 |
|------|------|------|
| **容器 Containers** | 存储数据的结构 | 锅碗瓢盆（存东西） |
| **迭代器 Iterators** | 访问容器元素的"指针" | 夹子（取东西） |
| **算法 Algorithms** | 对容器进行操作 | 菜谱（怎么做） |

**为什么竞赛中要学 STL？**

1. **提高编码效率**：手写一个平衡树要上百行，用 `set` 只要一行声明
2. **减少 bug**：标准库经过严格测试，比手写更可靠
3. **专注算法本身**：把"造轮子"的时间省下来想算法
4. **考场允许使用**：CSP-J/S 全部允许使用 STL

```cpp
#include <bits/stdc++.h>   // 万能头文件，包含所有 STL
using namespace std;       // 使用标准命名空间，省去 std:: 前缀
```

!!! info "关于万能头文件"
    `#include <bits/stdc++.h>` 包含了几乎所有标准库头文件（vector、set、map、algorithm 等），竞赛中统一用它最省事。注意：部分非 GCC 编译器（如 MSVC）不支持此头文件，但 CSP 考场用的是 GCC，可放心使用。

---

### A.2 序列容器

序列容器按**线性顺序**存储元素，元素的位置由插入顺序决定。

#### A.2.1 vector（动态数组）

`vector` 是最常用的容器，本质是**能自动扩容的数组**。它兼顾了数组的随机访问效率与链表的动态大小优点。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `v.push_back(x)` | 尾部添加元素 | O(1) 均摊 |
| `v.pop_back()` | 删除尾部元素 | O(1) |
| `v.size()` | 元素个数 | O(1) |
| `v.empty()` | 是否为空 | O(1) |
| `v.clear()` | 清空 | O(n) |
| `v.resize(n)` | 改变大小 | O(n) |
| `v[i]` | 随机访问第 i 个 | O(1) |
| `v.begin()` / `v.end()` | 首尾迭代器 | O(1) |
| `v.insert(it, x)` | 在迭代器处插入 | O(n) |
| `v.erase(it)` | 删除迭代器处元素 | O(n) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v;
    v.push_back(3);          // v = {3}
    v.push_back(1);          // v = {3, 1}
    v.push_back(4);          // v = {3, 1, 4}
    v.push_back(1);          // v = {3, 1, 4, 1}
    v.push_back(5);          // v = {3, 1, 4, 1, 5}

    cout << v.size() << "\n";        // 5
    cout << v[0] << "\n";            // 3（下标从 0 开始）

    // 范围 for 循环遍历
    for (int x : v) cout << x << " ";
    cout << "\n";                    // 3 1 4 1 5

    v.pop_back();             // 删除尾部，v = {3, 1, 4, 1}
    sort(v.begin(), v.end()); // 排序，v = {1, 1, 3, 4}

    // 二维 vector（动态二维数组）
    vector<vector<int>> grid(3, vector<int>(4, 0));  // 3行4列，初始0
    grid[1][2] = 7;
    return 0;
}
```

!!! warning "常见误区：遍历时修改 vector 导致迭代器失效"
    ```cpp
    // ❌ 错误：边遍历边删除，迭代器失效
    for (auto it = v.begin(); it != v.end(); it++) {
        if (*it == 1) v.erase(it);   // it 已失效，++it 崩溃！
    }
    // ✅ 正确：用 erase 返回的迭代器
    for (auto it = v.begin(); it != v.end(); ) {
        if (*it == 1) it = v.erase(it);
        else it++;
    }
    ```

!!! tip "竞赛常用：vector 存图（邻接表）"
    ```cpp
    vector<int> adj[N];           // adj[u] 存 u 的所有邻居
    adj[1].push_back(2);          // 1 → 2 的边
    adj[1].push_back(3);          // 1 → 3 的边
    for (int v : adj[1]) { ... }  // 遍历 1 的所有邻居
    ```
    这是 CSP-S 中存图的标准写法，比手写链式前向星更简洁（但常数略大）。

#### A.2.2 string（字符串）

`string` 是专门处理字符序列的容器，比 C 风格的 `char[]` 更安全、更方便。

| 操作 | 含义 | 示例 |
|------|------|------|
| `s.size()` / `s.length()` | 长度 | `"abc".size()` → 3 |
| `s[i]` | 下标访问 | `s[0]` → 首字符 |
| `s.substr(pos, len)` | 子串 | `"abcdef".substr(1,3)` → "bcd" |
| `s.find(t)` | 查找子串 | 返回下标，找不到返回 `string::npos` |
| `s += t` | 追加 | `s += "xyz"` |
| `s.replace(pos, len, t)` | 替换 | —— |
| `s.c_str()` | 转 C 风格字符串 | 用于 `printf("%s", s.c_str())` |
| `+` | 拼接 | `"ab" + "cd"` → "abcd" |
| `==, <, >` | 比较 | 按字典序比较 |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s = "Hello";
    string t = "World";

    cout << s.size() << "\n";          // 5
    cout << s + " " + t << "\n";       // Hello World
    cout << s.substr(1, 3) << "\n";    // ell

    // find：查找子串
    string s2 = "abcabc";
    cout << s2.find("bc") << "\n";     // 1（第一次出现位置）
    if (s2.find("xyz") == string::npos)  // 找不到
        cout << "not found\n";

    // 比较：按字典序
    cout << ("apple" < "banana") << "\n";  // 1（true）
    cout << ("apple" < "apricot") << "\n"; // 1（'p' < 'r'）

    // getline 读整行（含空格）
    string line;
    getline(cin, line);                // 读一整行，含空格
    cout << line << "\n";
    return 0;
}
```

!!! warning "cin >> s 读不了带空格的字符串"
    `cin >> s` 遇到**空格就停止**。输入 `Hello World`，`s` 只会存 `Hello`。
    想读含空格的整行，用 `getline(cin, s)`。
    注意：`cin >> n;` 后接 `getline()` 时，要先用 `cin.ignore()` 吃掉残留的换行符。

```cpp
int n;
cin >> n;
cin.ignore();                 // 吃掉 n 后面的换行符，否则 getline 会读到空行
string line;
getline(cin, line);
```

#### A.2.3 pair（二元组）

`pair` 把两个值"绑"在一起，常用于存储键值对、坐标、带权边等。

| 操作 | 含义 |
|------|------|
| `make_pair(a, b)` | 创建 pair |
| `p.first` | 第一个值 |
| `p.second` | 第二个值 |
| `==, <, >` | 比较（先比 first，再比 second） |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    pair<int, int> p1(3, 5);
    pair<int, int> p2 = make_pair(1, 2);
    auto p3 = make_pair(1, 3);          // auto 自动推导类型

    cout << p1.first << " " << p1.second << "\n";   // 3 5

    // pair 比较规则：先比 first，first 相同再比 second
    cout << (p2 < p3) << "\n";          // 1（p2.first==p3.first==1，p2.second=2<3）

    // 常见用途 1：存坐标
    pair<int, int> point(3, 4);

    // 常见用途 2：存带权边（边权，终点），sort 后自动按边权排序
    vector<pair<int, int>> edges;
    edges.push_back({5, 2});            // 边权5，终点2（C++11 列表初始化）
    edges.push_back({2, 3});
    edges.push_back({5, 1});
    sort(edges.begin(), edges.end());   // 按 first（边权）排序，相同再按 second

    // 常见用途 3：函数返回两个值
    auto result = make_pair(42, "hello");
    cout << result.first << " " << result.second << "\n";
    return 0;
}
```

!!! tip "pair 与 sort 配合的经典用法"
    把边存成 `pair<int,int>`，`first` 存权值，`second` 存终点。`sort` 后自动按权值排序，这是 Kruskal 最小生成树算法的核心步骤。

---

### A.3 容器适配器

容器适配器是对底层容器的**封装**，只暴露特定接口。它们不能直接遍历。

#### A.3.1 stack（栈）

栈是**后进先出（LIFO）**结构，详见第18章。这里展示 STL 版本。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `s.push(x)` | 入栈 | O(1) |
| `s.pop()` | 出栈（删除栈顶，不返回值） | O(1) |
| `s.top()` | 访问栈顶元素（不删除） | O(1) |
| `s.empty()` | 是否为空 | O(1) |
| `s.size()` | 元素个数 | O(1) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    stack<int> s;
    s.push(1);          // 栈：[1]
    s.push(2);          // 栈：[1, 2]
    s.push(3);          // 栈：[1, 2, 3]

    cout << s.top() << "\n";   // 3（栈顶）
    s.pop();                   // 栈：[1, 2]
    cout << s.top() << "\n";   // 2

    // 遍历栈：必须边 pop 边访问（stack 不支持迭代器）
    while (!s.empty()) {
        cout << s.top() << " ";   // 2 1
        s.pop();
    }
    return 0;
}
```

!!! warning "s.pop() 不返回值"
    `pop()` 只删除栈顶，**不返回它**。要获取栈顶值必须先 `top()` 再 `pop()`：
    ```cpp
    int x = s.top();   // 先取值
    s.pop();           // 再删除
    ```
    对空栈调用 `top()` 或 `pop()` 是未定义行为，可能崩溃，务必先检查 `empty()`。

#### A.3.2 queue（队列）

队列是**先进先出（FIFO）**结构，详见第19章。BFS 的核心数据结构。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `q.push(x)` | 入队（队尾） | O(1) |
| `q.pop()` | 出队（删除队首，不返回值） | O(1) |
| `q.front()` | 访问队首元素 | O(1) |
| `q.back()` | 访问队尾元素 | O(1) |
| `q.empty()` | 是否为空 | O(1) |
| `q.size()` | 元素个数 | O(1) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    queue<int> q;
    q.push(10);         // 队列：[10]
    q.push(20);         // 队列：[10, 20]
    q.push(30);         // 队列：[10, 20, 30]

    cout << q.front() << "\n";   // 10（队首）
    cout << q.back() << "\n";    // 30（队尾）
    q.pop();                     // 队列：[20, 30]
    cout << q.front() << "\n";   // 20
    return 0;
}
```

!!! tip "BFS 标准模板"
    ```cpp
    queue<int> q;
    q.push(start);
    visited[start] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
    ```

#### A.3.3 priority_queue（优先队列 = 堆）

优先队列是一种特殊的队列：**每次出队的是最大（或最小）的元素**，本质是**堆**。详见第22章。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `pq.push(x)` | 插入元素 | O(log n) |
| `pq.pop()` | 删除堆顶（不返回值） | O(log n) |
| `pq.top()` | 访问堆顶元素 | O(1) |
| `pq.empty()` | 是否为空 | O(1) |
| `pq.size()` | 元素个数 | O(1) |

**默认是大根堆**（堆顶最大）。定义小根堆需要三个模板参数：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    // 大根堆（默认）
    priority_queue<int> maxHeap;
    maxHeap.push(3); maxHeap.push(1); maxHeap.push(4);
    cout << maxHeap.top() << "\n";    // 4（最大值在堆顶）

    // 小根堆：三个参数缺一不可
    priority_queue<int, vector<int>, greater<int>> minHeap;
    minHeap.push(3); minHeap.push(1); minHeap.push(4);
    cout << minHeap.top() << "\n";    // 1（最小值在堆顶）
    return 0;
}
```

!!! warning "小根堆的三个参数"
    定义小根堆必须写全三个参数：`priority_queue<int, vector<int>, greater<int>>`。
    - 第一个 `int`：元素类型
    - 第二个 `vector<int>`：底层容器（必须是 vector）
    - 第三个 `greater<int>`：比较方式（greater = 小的优先 = 小根堆）
    注意 `>>` 之间要有空格（C++11 之前 `>>` 会被当成右移运算符）。

**自定义比较**（用于存结构体的优先队列）：

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int dist, id;
    // 方法1：重载 < 运算符（priority_queue 默认用 < 构建，< 小的优先级低在堆底）
    // 想让 dist 小的优先（小根堆），要让 dist 小的 "更大"
    bool operator < (const Node &o) const {
        return dist > o.dist;   // 注意方向：> 表示 dist 小的优先级高
    }
};

int main() {
    priority_queue<Node> pq;
    pq.push({5, 1});
    pq.push({2, 2});
    pq.push({8, 3});
    cout << pq.top().id << "\n";   // 2（dist=2 最小，先出队）
    return 0;
}
```

!!! tip "Dijkstra 堆优化必用小根堆"
    Dijkstra 最短路算法（第30章）用小根堆按距离排序：
    ```cpp
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    // pair 的比较：先比 first（距离），相同再比 second（节点）
    ```
    pair 配合 `greater` 天然实现按距离排序的小根堆。

---

### A.4 关联容器（有序）

有序关联容器基于**红黑树**实现，元素自动排序，操作 O(log n)。

#### A.4.1 set（集合）

`set` 存储**不重复**且**自动有序**的元素集合。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `s.insert(x)` | 插入元素 | O(log n) |
| `s.erase(x)` | 删除值为 x 的元素 | O(log n) |
| `s.find(x)` | 查找 x，返回迭代器 | O(log n) |
| `s.count(x)` | x 的个数（set 中只能 0 或 1） | O(log n) |
| `s.lower_bound(x)` | 第一个 >= x 的迭代器 | O(log n) |
| `s.upper_bound(x)` | 第一个 > x 的迭代器 | O(log n) |
| `s.size()` / `s.empty()` / `s.clear()` | 常规操作 | O(1) / O(1) / O(n) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    set<int> s;
    s.insert(3); s.insert(1); s.insert(4); s.insert(1);  // 重复的1自动忽略
    // s = {1, 3, 4}，自动排序去重

    cout << s.size() << "\n";      // 3

    // 判断元素是否存在
    if (s.count(3)) cout << "3 exists\n";    // 1（存在）
    if (!s.count(5)) cout << "5 not found\n";

    // find 返回迭代器
    auto it = s.find(4);
    if (it != s.end()) cout << "found: " << *it << "\n";   // found: 4

    // 遍历（有序）
    for (int x : s) cout << x << " ";   // 1 3 4

    // lower_bound / upper_bound
    cout << *s.lower_bound(2) << "\n";  // 3（第一个 >= 2 的）
    cout << *s.upper_bound(3) << "\n";  // 4（第一个 > 3 的）
    return 0;
}
```

!!! warning "find() vs count() 的区别"
    - `find(x)` 返回**迭代器**，找不到返回 `s.end()`。适合"找到后还要操作"的场景。
    - `count(x)` 返回**整数**（0 或 1），适合"只判断是否存在"。
    - 两者复杂度都是 O(log n)，但 `count` 在 `multiset` 中可以返回重复个数。

#### A.4.2 multiset（多重集合）

`multiset` 与 `set` 几乎相同，唯一区别是**允许重复元素**。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    multiset<int> ms;
    ms.insert(3); ms.insert(1); ms.insert(3); ms.insert(3);
    // ms = {1, 3, 3, 3}

    cout << ms.count(3) << "\n";     // 3（有三个3）

    // erase(x) 会删除所有等于 x 的元素！
    ms.erase(3);                     // ms = {1}，所有3都被删了
    cout << ms.size() << "\n";       // 1

    // 只删一个：用 erase + find
    ms.insert(3); ms.insert(3);
    ms.erase(ms.find(3));            // 只删一个3
    cout << ms.count(3) << "\n";     // 1
    return 0;
}
```

!!! warning "multiset.erase(x) 会删除所有等于 x 的元素"
    `ms.erase(3)` 会删除**所有**值为 3 的元素。只想删一个，用 `ms.erase(ms.find(3))`。

#### A.4.3 map（映射）

`map` 存储**有序键值对**，按 key 自动排序，key 不重复。

| 操作 | 含义 | 复杂度 |
|------|------|--------|
| `mp[key]` | 访问/插入（key 不存在则插入默认值） | O(log n) |
| `mp.insert({k, v})` | 插入键值对 | O(log n) |
| `mp.erase(key)` | 删除 key | O(log n) |
| `mp.find(key)` | 查找 key，返回迭代器 | O(log n) |
| `mp.count(key)` | key 的个数（0 或 1） | O(log n) |
| `mp.size()` / `mp.empty()` | 常规操作 | O(1) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    map<string, int> mp;
    mp["apple"] = 3;
    mp["banana"] = 5;
    mp["cherry"] = 2;

    // 访问
    cout << mp["apple"] << "\n";      // 3

    // 遍历（按 key 字典序）
    for (auto &p : mp) {
        cout << p.first << ": " << p.second << "\n";
    }
    // 输出（有序）：
    // apple: 3
    // banana: 5
    // cherry: 2

    // 判断 key 是否存在
    if (mp.count("grape")) { ... }    // 0，不存在
    return 0;
}
```

!!! warning "mp[] 会插入默认值"
    `mp[key]` 在 key **不存在时会自动插入**一个默认值（int 为 0，string 为 ""）。
    ```cpp
    map<string, int> mp;
    cout << mp["abc"] << "\n";   // 输出 0，但 mp 里现在多了一个 "abc"->0！
    cout << mp.size() << "\n";   // 1（被意外插入了）
    ```
    **正确做法**：只判断是否存在用 `count()` 或 `find()`，不要用 `[]`：
    ```cpp
    if (mp.count("abc")) cout << mp["abc"];   // 先判断再访问
    ```

---

### A.5 关联容器（无序）

`unordered_set` 和 `unordered_map` 基于**哈希表**实现，平均 O(1) 但无序。

| 对比项 | set / map | unordered_set / unordered_map |
|--------|-----------|-------------------------------|
| 底层结构 | 红黑树 | 哈希表 |
| 是否有序 | 有序 | 无序 |
| 查找/插入复杂度 | O(log n) | O(1) 期望（最坏 O(n)） |
| 支持 lower_bound | 是 | 否 |
| 内存占用 | 较小 | 较大（哈希表开销） |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    unordered_map<string, int> ump;
    ump["apple"] = 3;
    ump["banana"] = 5;

    cout << ump["apple"] << "\n";      // 3
    cout << ump.count("banana") << "\n"; // 1

    // 遍历（无序！顺序不确定）
    for (auto &p : ump) {
        cout << p.first << " " << p.second << "\n";
    }
    return 0;
}
```

!!! tip "set vs unordered_set 选择原则"
    - 需要**有序**遍历或 `lower_bound`/`upper_bound` → 用 `set`/`map`
    - 只需要**快速查找/插入**，不关心顺序 → 用 `unordered_set`/`unordered_map`（更快）
    - 竞赛中，如果不需要有序性，优先选 unordered 版本（常数更小）

!!! warning "自定义类型作为 unordered_map 的 key"
    `unordered_map` 的 key 必须有哈希函数。`int`、`string` 等内置类型自带哈希，但**自定义结构体**需要自己实现哈希函数，否则编译错误。而 `map` 只需要重载 `<`，更简单。所以自定义类型优先用 `map`。

---

### A.6 迭代器与范围 for 循环

**迭代器（Iterator）** 是访问容器元素的通用"指针"，让算法能以统一方式操作不同容器。

```cpp
vector<int> v = {3, 1, 4, 1, 5};

// 迭代器遍历
for (vector<int>::iterator it = v.begin(); it != v.end(); it++) {
    cout << *it << " ";        // *it 解引用
}

// auto 简化（C++11）
for (auto it = v.begin(); it != v.end(); it++) {
    cout << *it << " ";
}

// 范围 for 循环（最简洁，推荐）
for (int x : v) {
    cout << x << " ";
}

// 反向迭代器
for (auto it = v.rbegin(); it != v.rend(); it++) {
    cout << *it << " ";        // 逆序输出
}
```

| 迭代器 | 含义 |
|--------|------|
| `v.begin()` | 指向第一个元素 |
| `v.end()` | 指向**最后一个元素的下一个**（不指向有效元素） |
| `v.rbegin()` | 反向起始（指向最后一个元素） |
| `v.rend()` | 反向终点（指向第一个元素的前一个） |

!!! info "范围 for 循环"
    `for (int x : v)` 等价于用迭代器从头遍历到尾，是最推荐的遍历方式。需要修改元素时用引用：`for (int &x : v) x *= 2;`

---

### A.7 常用算法（<algorithm>）

`<algorithm>` 头文件提供了大量通用算法，竞赛中最常用的如下。

#### A.7.1 sort（排序）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    // 对数组排序
    int a[] = {5, 2, 8, 1, 9};
    int n = 5;
    sort(a, a + n);                    // 升序：1 2 5 8 9
    sort(a, a + n, greater<int>());    // 降序：9 8 5 2 1

    // 对 vector 排序
    vector<int> v = {5, 2, 8, 1, 9};
    sort(v.begin(), v.end());          // 升序

    // 自定义比较函数
    vector<pair<int,int>> vp = {{3,2}, {1,5}, {3,1}};
    // 按 first 升序，first 相同按 second 降序
    sort(vp.begin(), vp.end(), [](const pair<int,int>& a, const pair<int,int>& b) {
        if (a.first != b.first) return a.first < b.first;
        return a.second > b.second;
    });
    return 0;
}
```

**自定义比较函数**（不用 lambda 的经典写法）：

```cpp
bool cmp(const int &a, const int &b) {
    return a > b;    // 降序
}
sort(v.begin(), v.end(), cmp);
```

!!! warning "cmp 必须满足严格弱序"
    比较函数 `cmp(a, b)` 返回 true 表示 a 应排在 b **前面**。必须满足：
    - `cmp(a, a)` 必须为 `false`（不能写 `<=`，要写 `<`）
    - 传递性：`cmp(a,b)` 且 `cmp(b,c)` 则 `cmp(a,c)`
    写成 `a <= b` 会导致 sort 出错甚至崩溃！

#### A.7.2 lower_bound / upper_bound（二分查找）

要求容器**有序**，在 O(log n) 时间内查找。

| 函数 | 含义 | 返回值 |
|------|------|--------|
| `lower_bound(begin, end, x)` | 第一个 **>= x** 的位置 | 迭代器 |
| `upper_bound(begin, end, x)` | 第一个 **> x** 的位置 | 迭代器 |

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a[] = {1, 2, 4, 4, 5, 7, 9};
    int n = 7;

    // 数组版本
    int *p = lower_bound(a, a + n, 4);
    cout << *p << " " << (p - a) << "\n";   // 4 2（值4，下标2）

    int *q = upper_bound(a, a + n, 4);
    cout << *q << " " << (q - a) << "\n";   // 5 4（值5，下标4）

    // vector 版本
    vector<int> v = {1, 2, 4, 4, 5, 7, 9};
    auto it = lower_bound(v.begin(), v.end(), 4);
    cout << *it << "\n";                    // 4

    // 统计 4 出现的次数 = upper_bound - lower_bound
    int cnt = upper_bound(v.begin(), v.end(), 4) - lower_bound(v.begin(), v.end(), 4);
    cout << cnt << "\n";                    // 2
    return 0;
}
```

!!! tip "二分查找的速记"
    - `lower_bound` = "下界" = 第一个 **≥** x（含等于）
    - `upper_bound` = "上界" = 第一个 **>** x（不含等于）
    - 两者之差 = x 出现的次数

#### A.7.3 其他常用算法

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {3, 1, 4, 1, 5};

    // 1. reverse：反转
    reverse(v.begin(), v.end());      // v = {5, 1, 4, 1, 3}

    // 2. unique：去重（需先排序！返回去重后的尾迭代器）
    sort(v.begin(), v.end());         // v = {1, 1, 3, 4, 5}
    auto last = unique(v.begin(), v.end());
    v.erase(last, v.end());          // v = {1, 3, 4, 5}，必须配合 erase

    // 3. max / min
    cout << max(3, 5) << "\n";        // 5
    cout << min(3, 5) << "\n";        // 3
    cout << max({1, 2, 3, 4}) << "\n"; // 4（多个值的最大值，C++11）

    // 4. swap
    int a = 3, b = 5;
    swap(a, b);                       // a=5, b=3

    // 5. fill：填充
    vector<int> v2(5);
    fill(v2.begin(), v2.end(), 42);   // v2 = {42, 42, 42, 42, 42}

    // 6. next_permutation：下一个全排列
    vector<int> p = {1, 2, 3};
    do {
        for (int x : p) cout << x << " ";
        cout << "\n";
    } while (next_permutation(p.begin(), p.end()));
    // 输出所有 6 种排列（1 2 3 / 1 3 2 / 2 1 3 / ...）

    // 7. __gcd：最大公约数（双下划线，GCC 内置）
    cout << __gcd(12, 18) << "\n";    // 6
    // C++17 起可用 gcd：cout << gcd(12, 18) << "\n";
    return 0;
}
```

!!! warning "unique 必须先排序"
    `unique` 只删除**相邻**的重复元素。要彻底去重，必须先 `sort` 再 `unique` 再 `erase`：
    ```cpp
    sort(v.begin(), v.end());
    v.erase(unique(v.begin(), v.end()), v.end());
    ```
    这三步是竞赛中"排序去重"的标准写法，请背下来。

---

## 代码实现

### 完整示例：STL 综合运用

下面用一个综合示例演示 STL 各组件的配合使用。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    // 场景：统计单词出现次数，按次数降序输出，次数相同按单词字典序
    int n;
    cin >> n;
    map<string, int> cnt;          // map 自动按 key(单词) 排序

    for (int i = 0; i < n; i++) {
        string word;
        cin >> word;
        cnt[word]++;               // 统计频次
    }

    // 转存到 vector<pair> 以便按频次排序
    vector<pair<int, string>> v;   // first 存频次，second 存单词
    for (auto &p : cnt) {
        v.push_back({p.second, p.first});
    }

    // 按频次降序，频次相同按单词升序
    sort(v.begin(), v.end(), [](const pair<int,string>& a, const pair<int,string>& b) {
        if (a.first != b.first) return a.first > b.first;   // 频次降序
        return a.second < b.second;                          // 单词升序
    });

    // 输出结果
    for (auto &p : v) {
        cout << p.second << " " << p.first << "\n";
    }

    // 用 set 统计不同单词数
    set<string> unique_words;
    for (auto &p : cnt) unique_words.insert(p.first);
    cout << "共 " << unique_words.size() << " 个不同单词\n";

    // 用 priority_queue 找出现次数最多的单词
    priority_queue<pair<int,string>> pq;   // 大根堆，按频次排序
    for (auto &p : cnt) pq.push({p.second, p.first});
    cout << "最频繁: " << pq.top().second << " (" << pq.top().first << "次)\n";
    return 0;
}
```

输入：
```
8
apple banana apple cherry banana apple date cherry
```
输出：
```
apple 3
banana 2
cherry 2
date 1
共 4 个不同单词
最频繁: apple (3次)
```

---

## ⚠️ 常见误区

### 误区 1：遍历时删除 vector 元素导致迭代器失效

```cpp
// ❌ 错误
for (auto it = v.begin(); it != v.end(); it++) {
    if (*it == target) v.erase(it);   // it 失效！
}
// ✅ 正确：用 erase 返回的新迭代器
for (auto it = v.begin(); it != v.end(); ) {
    if (*it == target) it = v.erase(it);
    else it++;
}
```

### 误区 2：用 map[] 判断 key 是否存在

```cpp
// ❌ 错误：[] 会插入默认值
if (mp["key"] == 0) { ... }   // "key" 不存在时被插入 0，mp 被修改
// ✅ 正确：用 count 或 find
if (mp.count("key")) { ... }
if (mp.find("key") != mp.end()) { ... }
```

### 误区 3：priority_queue 默认就是小根堆

```cpp
// ❌ 误以为默认小根堆
priority_queue<int> pq;        // 其实是大根堆！top() 是最大值
// ✅ 小根堆要写全三个参数
priority_queue<int, vector<int>, greater<int>> pq;
```

### 误区 4：multiset.erase(x) 删除所有等于 x 的元素

```cpp
multiset<int> ms = {1, 3, 3, 3, 5};
// ❌ 想删一个 3，结果全删了
ms.erase(3);                   // ms = {1, 5}
// ✅ 只删一个：用 find 取迭代器
ms.erase(ms.find(3));          // ms = {1, 3, 3, 5}
```

### 误区 5：对 list 用 std::sort

```cpp
list<int> lst = {3, 1, 4};
// ❌ sort 不支持 list（list 迭代器不是随机访问）
sort(lst.begin(), lst.end());           // 编译错误
// ✅ list 有自己的 sort 成员函数
lst.sort();                              // 正确
```

### 误区 6：忘记 sort 的比较函数要严格弱序

```cpp
// ❌ 错误：<= 不满足严格弱序
bool cmp(int a, int b) { return a <= b; }   // cmp(a,a)=true，违规
// ✅ 正确：用 <
bool cmp(int a, int b) { return a < b; }
```

### 误区 7：set/map 中修改 key

`set` 的元素和 `map` 的 key 在容器中是**不可修改**的（因为修改会破坏红黑树的有序性）。需要修改时，先 `erase` 再 `insert`。

---

## 典型例题

### 例题 1：[P1177] 快速排序模板（sort 练习）

**题意**：输入 n 个整数，用快速排序排序后输出。

**分析**：直接用 `sort` 即可，这是 sort 的模板题。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    scanf("%d", &n);
    vector<int> a(n);
    for (int i = 0; i < n; i++) scanf("%d", &a[i]);
    sort(a.begin(), a.end());          // STL 快排，O(n log n)
    for (int i = 0; i < n; i++)
        printf("%d%c", a[i], i == n-1 ? '\n' : ' ');
    return 0;
}
```

**复杂度**：时间 O(n log n)，空间 O(n)。

---

### 例题 2：[P2249] 查找（lower_bound 练习）

**题意**：给定一个严格递增的正整数序列，m 次询问，每次查询某个数第一次出现的位置（下标从 1 开始），不存在输出 -1。

**分析**：序列有序，用 `lower_bound` 二分查找。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    vector<int> a(n);
    for (int i = 0; i < n; i++) scanf("%d", &a[i]);
    while (m--) {
        int x;
        scanf("%d", &x);
        int pos = lower_bound(a.begin(), a.end(), x) - a.begin();
        if (pos < n && a[pos] == x) printf("%d ", pos + 1);  // 找到
        else printf("-1 ");                                    // 不存在
    }
    return 0;
}
```

**复杂度**：每次查询 O(log n)，总共 O(m log n)。

!!! tip "lower_bound 找到后要验证"
    `lower_bound` 返回的是第一个 **>= x** 的位置，不一定是 x 本身。必须检查 `a[pos] == x` 才能确认找到了。

---

### 例题 3：[P3378] 堆模板（priority_queue 练习）

**题意**：维护一个小根堆，支持两种操作：① 插入一个数；② 输出并删除最小值。

**分析**：直接用 `priority_queue` 的小根堆。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    scanf("%d", &n);
    // 小根堆
    priority_queue<int, vector<int>, greater<int>> pq;
    while (n--) {
        int op, x;
        scanf("%d", &op);
        if (op == 1) {
            scanf("%d", &x);
            pq.push(x);
        } else {
            printf("%d\n", pq.top());   // 输出最小值
            pq.pop();                   // 删除
        }
    }
    return 0;
}
```

**复杂度**：每次操作 O(log n)，总共 O(n log n)。

---

### 例题 4：[P2580] 于是他错误的点名开始了（map 练习）

**题意**：先给一个单词表（n 个单词），再给 m 次点名，每次判断：单词在表中且第一次点名 → "OK"；在表中但重复点名 → "REPEAT"；不在表中 → "WRONG"。

**分析**：用 `map<string, int>` 记录每个单词被点名的次数。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    string s;
    cin >> n;
    set<string> dict;                  // 存单词表
    map<string, int> called;           // 记录点名次数
    for (int i = 0; i < n; i++) {
        cin >> s;
        dict.insert(s);
    }
    cin >> m;
    while (m--) {
        cin >> s;
        if (!dict.count(s)) {
            cout << "WRONG\n";
        } else if (called[s] == 0) {
            called[s] = 1;
            cout << "OK\n";
        } else {
            cout << "REPEAT\n";
        }
    }
    return 0;
}
```

**复杂度**：每次查询 O(log n)。

---

## 📚 本节练习

### 🟢 基础题（STL 基本操作）

**练习 1**：[P1177] 快速排序模板 - https://www.luogu.com.cn/problem/P1177
> 用 `sort` 对 n 个数排序。提示：注意数据范围，用 `scanf/printf` 或关闭同步。

**练习 2**：[P2249] 查找 - https://www.luogu.com.cn/problem/P2249
> 在有序序列中二分查找。提示：`lower_bound` + 验证。

**练习 3**：[P3378] 堆模板 - https://www.luogu.com.cn/problem/P3378
> 小根堆的插入与取最小值。提示：`priority_queue<int, vector<int>, greater<int>>`。

### 🟡 进阶题（STL 综合应用）

**练习 4**：[P2580] 于是他错误的点名开始了 - https://www.luogu.com.cn/problem/P2580
> 用 `map` 或 `unordered_map` 判断单词状态。提示：注意区分"不在表中"和"重复点名"。

**练习 5**：[P3370] 字符串哈希 - https://www.luogu.com.cn/problem/P3370
> 统计不同字符串的个数。提示：用 `set<string>` 或 `unordered_set<string>` 自动去重。

**练习 6**：[P1059] 明明的随机数 - https://www.luogu.com.cn/problem/P1059
> 去重并排序。提示：`sort` + `unique` + `erase` 三件套，或直接用 `set`。

### 🔴 综合题（STL 与算法结合）

**练习 7**：[P1090] 合并果子 - https://www.luogu.com.cn/problem/P1090
> 每次合并最小的两堆，求最小代价。提示：小根堆 `priority_queue`，每次取两个最小合并后放回。

**练习 8**：[P1631] 序列合并 - https://www.luogu.com.cn/problem/P1631
> 两个有序序列，各取一个合并，求前 n 小的和。提示：小根堆 + 去重，经典多路归并。

---

## 📖 本专题总结

### 思维导图

```
                    ┌──────────────────────┐
                    │   STL 标准模板库     │
                    └──────────┬───────────┘
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
       容器               迭代器                算法(algorithm)
          │              begin/end                │
    ┌─────┼──────┐       范围for           ┌──────┼──────┐
    ▼     ▼      ▼                          ▼      ▼      ▼
  序列   适配器  关联                     sort   bound  unique
   │      │      │                      (nlogn) (二分)  (去重)
 vector  stack  set/multiset           reverse max min swap
 string  queue  map                    fill  next_permutation
 pair    pqueue unordered_set/map
```

### STL 速查总表

| 分类 | 容器 | 头文件 | 特点 | 核心复杂度 |
|------|------|--------|------|-----------|
| 序列 | `vector` | `<vector>` | 动态数组，尾部快 | O(1) 尾部操作 |
| 序列 | `string` | `<string>` | 字符串 | O(1) 尾部操作 |
| 序列 | `pair` | `<utility>` | 二元组 | O(1) |
| 适配器 | `stack` | `<stack>` | LIFO 栈 | O(1) |
| 适配器 | `queue` | `<queue>` | FIFO 队列 | O(1) |
| 适配器 | `priority_queue` | `<queue>` | 堆（默认大根） | O(log n) |
| 有序关联 | `set` | `<set>` | 有序不重复集合 | O(log n) |
| 有序关联 | `multiset` | `<set>` | 有序可重复集合 | O(log n) |
| 有序关联 | `map` | `<map>` | 有序键值对 | O(log n) |
| 无序关联 | `unordered_set` | `<unordered_set>` | 哈希不重复 | O(1) 期望 |
| 无序关联 | `unordered_map` | `<unordered_map>` | 哈希键值对 | O(1) 期望 |

### 常见误区速查表

| 误区 | 正确做法 | 后果 |
|------|----------|------|
| 遍历时删除 vector 元素 | 用 `erase()` 返回的迭代器 | 迭代器失效崩溃 |
| `map[]` 用于查找 | 用 `find()` 或 `count()` | 意外插入元素 |
| set/map 修改 key | 删除后重新插入 | 破坏有序性 |
| `sort` 用于 list | list 用 `sort()` 成员函数 | 编译错误 |
| `cmp` 用 `<=` | 必须用 `<`（严格弱序） | sort 出错/崩溃 |
| `multiset.erase(x)` 想删一个 | 用 `erase(find(x))` | 删了所有等于 x 的 |
| `priority_queue` 默认大根堆当小根堆用 | 加 `greater<int>` | 得到错误结果 |
| `unique` 不先排序 | 先 `sort` 再 `unique` 再 `erase` | 去重不彻底 |
| 自定义类型作 unordered_map 的 key | 提供哈希函数或改用 map | 编译错误 |
| `cin >> s` 读含空格字符串 | 用 `getline(cin, s)` | 只读到空格前 |

### 核心口诀

- ✅ **vector** 动态数组，`push_back` 尾加，下标随机访问 O(1)
- ✅ **stack** 后进先出，`top` 取栈顶，`pop` 只删不返回
- ✅ **queue** 先进先出，`front` 取队首，BFS 必备
- ✅ **priority_queue** 默认大根堆，小根堆加 `greater<int>`
- ✅ **set** 有序不重复，`insert` 自动排序去重
- ✅ **map** 有序键值对，`[]` 会插入默认值，查找用 `count`
- ✅ **unordered** 哈希实现 O(1)，不有序但更快
- ✅ **sort** O(n log n)，比较函数用 `<` 不用 `<=`
- ✅ **lower_bound** 找 ≥x，**upper_bound** 找 >x，要求有序
- ✅ **排序去重三件套**：`sort` → `unique` → `erase`

### STL 选择决策树

```
需要存什么？
├─ 一组元素（线性）
│   ├─ 需要下标随机访问 → vector
│   └─ 频繁头尾操作 → deque
├─ 一组元素（后进先出/先进先出）
│   ├─ 后进先出 → stack
│   ├─ 先进先出 → queue
│   └─ 按优先级出 → priority_queue（堆）
├─ 一组元素（去重/查找）
│   ├─ 需要有序 → set / multiset
│   └─ 不需要有序 → unordered_set
├─ 键值对
│   ├─ 需要按 key 有序 → map
│   └─ 不需要有序 → unordered_map
└─ 两个值绑定 → pair
```

---

!!! tip "后续衔接"
    本专题学完后，你已掌握 STL 这把"瑞士军刀"。在第27章 DFS 进阶中，你会用 `vector` 存图、用 `stack` 实现迭代式 DFS；在第30章 Dijkstra 中，你会用 `priority_queue` 实现堆优化；在第33章 Kruskal 中，你会用 `sort` + `pair` 对边排序。STL 是后续所有算法章节的基础工具，请务必熟练掌握。
