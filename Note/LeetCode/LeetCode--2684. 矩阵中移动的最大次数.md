[2684. 矩阵中移动的最大次数](https://leetcode.cn/problems/maximum-number-of-moves-in-a-grid/)

> 给你一个下标从 **0** 开始、大小为 `m x n` 的矩阵 `grid` ，矩阵由若干 **正** 整数组成。
>
> 你可以从矩阵第一列中的 **任一** 单元格出发，按以下方式遍历 `grid` ：
>
> - 从单元格 `(row, col)` 可以移动到 `(row - 1, col + 1)`、`(row, col + 1)` 和 `(row + 1, col + 1)` 三个单元格中任一满足值 **严格** 大于当前单元格的单元格。
>
> 返回你在矩阵中能够 **移动** 的 **最大** 次数。

---

💩题目，非要把遍历顺序反过来，给我卡了半天

```go
func maxMoves(grid [][]int) int {
    n, m := len(grid), len(grid[0])
    ans := 0
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m)
        for j := 0; j < m; j ++ {
            f[i][j] = -1
        }
    }
    for i := 0; i < n; i ++ {
        f[i][0] = 0
    }
    for i := 1; i < m; i ++ {
        for j := 0; j < n; j ++ {
            if grid[j][i] > grid[j][i - 1] && f[j][i - 1] != -1 {
                f[j][i] = f[j][i - 1] + 1
            } else if j + 1 < n && grid[j][i] > grid[j + 1][i - 1] && f[j + 1][i - 1] != -1 {
                f[j][i] = f[j + 1][i - 1] + 1
            } else if j - 1 >= 0 && grid[j][i] > grid[j - 1][i - 1] && f[j - 1][i - 1] != -1 {
                f[j][i] = f[j - 1][i - 1] + 1
            }
            ans = max(ans, f[j][i])
        }
    }
    return ans
}
```



