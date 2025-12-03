[2435. 矩阵中和能被 K 整除的路径](https://leetcode.cn/problems/paths-in-matrix-whose-sum-is-divisible-by-k/)

> 给你一个下标从 **0** 开始的 `m x n` 整数矩阵 `grid` 和一个整数 `k` 。你从起点 `(0, 0)` 出发，每一步只能往 **下** 或者往 **右** ，你想要到达终点 `(m - 1, n - 1)` 。
>
> 请你返回路径和能被 `k` 整除的路径数目，由于答案可能很大，返回答案对 `109 + 7` **取余** 的结果。

---

感觉相比前面一道 hard（[1301. 最大得分的路径数目](https://leetcode.cn/problems/number-of-paths-with-max-score/)）还是要简单很多，这道题重点在于如何找准状态转移方程，很显然，这道题应该使用三维数组，而不是使用另一个二维数组来记录路径和，此时并没有要求我们记录最大路径和，所以如果使用二维数组会很困难，最好是使用三维数组来进行状态转移：

```c
func numberOfPaths(grid [][]int, k int) int {
    mod := 1000000007
    n, m := len(grid), len(grid[0])
    f := make([][][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i]= make([][]int, m + 1)
        for j := 0; j <= m; j ++ {
            f[i][j] = make([]int, k)
        }
    }
    f[0][1][0] = 1
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            for t := 0; t < k; t ++ {
                f[i + 1][j + 1][(t + grid[i][j]) % k] = (f[i + 1][j][t] + f[i][j + 1][t]) % mod
            }
        }
    }
    return f[n][m][0]
}
```

