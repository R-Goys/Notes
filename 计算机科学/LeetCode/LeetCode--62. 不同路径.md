[62. 不同路径](https://leetcode.cn/problems/unique-paths/)

----



```go
func uniquePaths(m int, n int) int {
    f := make([][]int, m + 1)
    for i := 0; i <= m; i ++ {
        f[i] = make([]int, n + 1)
    }
    f[0][1] = 1
    for i := 1; i <= m; i ++ {
        for j := 1; j <= n; j ++ {
            f[i][j] = f[i - 1][j] + f[i][j - 1]
        }
    }
    return f[m][n]
}
```

太easy了，最简单的dp，贴个代码，懒得写题解



二刷

```go
func uniquePaths(m int, n int) int {
    f := make([][]int, m + 1)
    for i := 0; i <= m; i ++ {
        f[i] = make([]int, n + 1)
    }
    f[1][0] = 1
    for i := 1; i <= m; i ++ {
        for j := 1; j <= n; j ++ {
            f[i][j] = f[i][j - 1] + f[i - 1][j]
        }
    }
    return f[m][n]
}
```

直接秒了。



三刷：

```go
func uniquePaths(m int, n int) int {
    f := make([][]int, m + 1)
    for i := 0; i <= m; i ++ {
        f[i] = make([]int, n + 1)
    }
    f[1][1] = 1
    for i := 1; i <= m; i ++ {
        for j := 1; j <= n; j ++ {
            f[i][j] += f[i - 1][j] + f[i][j - 1]
        }
    }
    return f[m][n]
}
```

