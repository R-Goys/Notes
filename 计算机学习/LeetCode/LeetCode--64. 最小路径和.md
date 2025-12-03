[64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)

----

dp题，dfs能做但是会超时，不如动态规划一根，这道题只需要理清楚f和grid的关系(画图帮助更大)即可，剩下都是很简单的。

无需多言，贴个题解。



```go
func minPathSum(grid [][]int) int {
    m := len(grid)
    n := len(grid[0])
    f := make([][]int, m)
    for i := 0; i < m; i ++ {
        f[i] = make([]int, n)
    }
    f[0][0] = grid[0][0]

    for i := 1; i < m; i ++ {
        f[i][0] = f[i - 1][0] + grid[i][0]
    }
    for i := 1; i < n; i ++ {
        f[0][i] = f[0][i - 1] + grid[0][i]
    }

    for i := 1; i < m; i ++ {
        for j := 1; j < n; j ++ {
            f[i][j] = min(f[i - 1][j], f[i][j - 1]) + grid[i][j]
        }
    }
    return f[m - 1][n - 1]
}
```

### 二刷

```go
func minPathSum(grid [][]int) int {
    n, m := len(grid), len(grid[0])
    if n == 0 && m == 0 {
        return 0
    }

    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m)
    }
    f[0][0] = grid[0][0]
    for i := 1; i < m; i ++ {
        f[0][i] = grid[0][i] + f[0][i - 1]
    }
    for i := 1; i < n; i ++ {
        f[i][0] = grid[i][0] + f[i - 1][0]
    }

    for i := 1; i < n; i ++ {
        for j := 1; j < m; j ++ {
            f[i][j] = min(f[i - 1][j], f[i][j - 1]) + grid[i][j]
        }
    }
    return f[n - 1][m - 1]
}
```

三刷：

```go
func minPathSum(grid [][]int) int {
    n, m := len(grid), len(grid[0])
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
        for j := 0; j <= m; j ++ {
            f[i][j] = 0x3f3f3f3f
        }
    }
    f[1][0], f[0][1] = 0, 0
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j] = min(f[i - 1][j], f[i][j - 1]) + grid[i - 1][j - 1]
        }
    }
    return f[n][m]
}
```

