[1594. 矩阵的最大非负积](https://leetcode.cn/problems/maximum-non-negative-product-in-a-matrix/)

> 给你一个大小为 `m x n` 的矩阵 `grid` 。最初，你位于左上角 `(0, 0)` ，每一步，你可以在矩阵中 **向右** 或 **向下** 移动。
>
> 在从左上角 `(0, 0)` 开始到右下角 `(m - 1, n - 1)` 结束的所有路径中，找出具有 **最大非负积** 的路径。路径的积是沿路径访问的单元格中所有整数的乘积。
>
> 返回 **最大非负积** 对 **`109 + 7`** **取余** 的结果。如果最大积为 **负数** ，则返回 `-1` 。
>
> **注意，**取余是在得到最大积之后执行的。

---

每个状态需要记录两个数字，最大值和最小值，和某一道题思路一样，遇到负数，最小值会变成最大值。

```go
func maxProductPath(grid [][]int) int {
    mod := 1000000007
    n, m := len(grid), len(grid[0])
    f_po := make([][]int, n)
    f_ne := make([][]int, n)
    for i := 0; i < n; i ++ {
        f_po[i] = make([]int, m)
        f_ne[i] = make([]int, m)
    }
    f_po[0][0], f_ne[0][0] = grid[0][0], grid[0][0]
    for i := 1; i < n; i ++ {
        f_po[i][0] = f_po[i - 1][0] * grid[i][0]
        f_ne[i][0] = f_ne[i - 1][0] * grid[i][0]
    }
    for i := 1; i < m; i ++ {
        f_po[0][i] = f_po[0][i - 1] * grid[0][i]
        f_ne[0][i] = f_ne[0][i - 1] * grid[0][i]
    }
    for i := 1; i < n; i ++ {
        for j := 1; j < m; j ++{
            f_po[i][j] = max(f_po[i - 1][j] * grid[i][j], f_po[i][j - 1] * grid[i][j], 
            f_ne[i - 1][j] * grid[i][j], f_ne[i][j - 1] * grid[i][j])

            f_ne[i][j] = min(f_po[i - 1][j] * grid[i][j], f_po[i][j - 1] * grid[i][j],
            f_ne[i - 1][j] * grid[i][j], f_ne[i][j - 1] * grid[i][j])
        }
    }
    if f_po[n - 1][m - 1] < 0 {
        return -1
    }
    return f_po[n - 1][m - 1] % mod
}
```

