 [1584. 连接所有点的最小费用](https://leetcode.cn/problems/min-cost-to-connect-all-points/)



此题是一道简单的最小生成树的模板题，

下面细说

# 题目分析

## Kruscal

题目给定了一些坐标（也就是点），由于每个坐标的x,y的组合都是独立的，为了方便在题目中我们使用一个编号来表示一个点。

让我们求得可以将所有坐标连通的最小的费用（两点之间的费用为`|xi - xj| + |yi - yj|` ）

想要入手这道题，首先要先找到每条边，以及每条边的费用。
 那么每条边可以通过结构体Edges来储存，其中l，r表示两个端点的编号，w表示连接两个端点的费用。

但是现在先不急，我们接着分析。

找到每两个端点的数值之后，我们可以通过排序将连接每条边的费用排序，并依次将两个集合合并（集合可以是一个点，也可以是已经连通的多个点形成的集合），当然，为了防止在已经连通的集合之中再建立一条边，我们可以在每次连接两个集合的同时，一个集合的祖宗节点更新为另一个集合的祖宗节点

### 祖宗节点

祖宗节点就是当前坐标通过已连接的边，寻根溯源找到的最开始的节点

那么回到刚刚的问题，为什么通过排序我们就能保证这样连接出来的费用是最少的呢？

那我们假设存在集合1，集合2，两个集合并未连通，并且在还未连接的边中（存在于集合内部的不算）不存在比新加入的边费用更小的边，那么我们可以判断，这条边是连接两个集合的最小费用。

于是思路如上

### 代码实现

下面我们可以来看看代码的实现：

```c++
class Solution {
public:
    int minCostConnectPoints(std::vector<std::vector<int>>& points) {
        // 定义边的结构体
        struct Edges {
            int l, r, w; // l和r是连接的点的索引，w是边的费用

            // 重载操作符，用于排序
            bool operator< (const Edges& other) const {
                return w < other.w; // 按照边的权重升序排序
            }
        };

        std::vector<Edges> Edge; // 用于存储所有边

        int F[1010][1010]; // 储存的数组 F（目前未使用）
        int idx = -1; // 边的索引
        int p[1010]; // 并查集数组，用于记录节点的父节点

        // 初始化并查集，每个节点指向自己，因为最开始没有能找的祖宗节点，所以只能是自己
        for(int i = 0; i < points.size(); i++) p[i] = i;

        // 生成所有的边及其权重
        for(int i = 0; i < points.size(); i++) {
            for(int j = i + 1; j < points.size(); j++) {
                // 计算两点之间的曼哈顿距离，并构建边
                Edge.push_back({i, j, abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1])});
            }
        }

        // 按照边的权重对边进行排序
        sort(Edge.begin(), Edge.end());

        int res = 0; // 结果，即最小生成树的总费用

        // 遍历每一条边，采用并查集判断是否在已连通的集合内重复建边
        for (int i = 0; i < Edge.size(); i++) {
            int a = Edge[i].l, b = Edge[i].r, w = Edge[i].w; // 获取边的两端点和费用
            int x = find(a, p), y = find(b, p); // 查找这两个顶点的祖宗节点
            if (x != y) { // 如果根节点不同，说明这条边不会形成环，即不存在于已连通的集合内
                p[x] = y; // 合并两个不同的集合，也相当于将两个集合的祖宗节点统一
                res += w; // 加入这条边的费用到总的费用
            }
        }
        return res; // 返回结果
    }

private:
    // 查找函数，带路径压缩
    int find(int x, int p[]) {
        if(x != p[x]) // 如果x不是根节点
            p[x] = find(p[x], p); // 递归查找并压缩路径
        return p[x]; // 返回根节点
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## Prim算法

至于这道题，还有另一种解法，那就是Prim算法，主要的思路是先将一个点加入集合之中，然后更新另外的每个点到这个集合的费用，然后选择费用最小的边连接的点加入这个集合，然后有通过min函数，不断遍历每个点，不断更新每个点到集合的最小值，思路简单，下面就是代码的实现

### 代码实现

```cpp
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        // 创建邻接矩阵 f 用于存储点之间的曼哈顿距离
        int f[1010][1010];
        
        // 创建一个大数组 dis，用于存储与最小生成树连接的点的最小距离
        int dis[1000010];
        
        // 初始化 dis 数组为无穷大（使用 0x3f3f3f3f表示无穷大）
        memset(dis, 0x3f, sizeof dis);
        
        // 初始化邻接矩阵 f 为 0
        memset(f, 0, sizeof f);

        // 计算所有点之间的费用
        for (int i = 0; i < points.size(); i++) {
            for (int j = i + 1; j < points.size(); j++) {
                // 费用的计算
                f[i][j] = abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1]);
                // 两点之间的距离不变，所以是无向图，故 f[j][i] 也应该等于 f[i][j]
                f[j][i] = f[i][j];
            }
        }

        // 获取点的数量
        int sz = points.size();

        // 调用 prim 算法计算最小生成树的总费用
        return prim(f, sz, dis);
    }

private:
    int prim(int f[1010][1010], int sz, int dis[]) {
        // 创建一个数组 st，用于标记点是否已被加入最小生成树
        int st[1010] = {0}; // 初始化为 0，表示所有点都未被访问
        
        // 最小生成树的总费用
        int ans = 0;

        // 先将第一个点加入集合，此时不需要费用，所以为0
        dis[0] = 0;

        // 遍历所有节点
        for (int i = 0; i < sz; i++) {
            int t = -1; // 用于记录距离最小的未访问节点

            // 找到当前未被访问的点中距离最小的点
            for (int j = 0; j < sz; j++) {
                // 如果当前点未被访问，且 t == -1 或更小的距离 found
                if (!st[j] && (t == -1 || dis[t] > dis[j])) {
                    t = j; // 更新 t 为当前的点 j
                }
            }

            // 如果所有点都已访问，跳出循环
            if (t == -1) break;

            // 将找到的最近的点 t 加入最小生成树
            ans += dis[t]; // 更新总权重
            st[t] = 1; // 标记 t 点为已访问

            // 更新其他点的最小费用
            for (int j = 0; j < sz; j++) {
                // 如果点 j 未被访问，尝试更新其最小距离
                if (!st[j]) {
                    dis[j] = min(f[t][j], dis[j]); // 比较当前点到原来集合的具体距离与通过 t 节点到达 j 的距离
                }
            }
        }

        // 返回最终计算的最小生成树的总费用
        return ans;
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 结语

这道题是一道简单的最小生成树的模板题，希望对你初步了解最小生成树有帮助，后面我会更新prim算法，尽情期待~

## -------END-------