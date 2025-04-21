[931. 下降路径最小和](https://leetcode.cn/problems/minimum-falling-path-sum/)

> 给你一个 `n x n` 的 **方形** 整数数组 `matrix` ，请你找出并返回通过 `matrix` 的**下降路径** 的 **最小和** 。
>
> **下降路径** 可以从第一行中的任何元素开始，并从每一行中选择一个元素。在下一行选择的元素和当前行所选元素最多相隔一列（即位于正下方或者沿对角线向左或者向右的第一个元素）。具体来说，位置 `(row, col)` 的下一个元素应当是 `(row + 1, col - 1)`、`(row + 1, col)` 或者 `(row + 1, col + 1)` 。

---

动态规划，这个边界问题还是比较好解决的，多开辟两层空间即可，初始化为0x3f3f3f3f即可

```go
func minFallingPathSum(matrix [][]int) int {
    n, m := len(matrix), len(matrix[0])
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m + 2)
        for j := 0; j <= m + 1; j ++ {
            f[i][j] = 0x3f3f3f3f
        }
    }
    for i := 1; i <= m; i ++ {
        f[0][i] = matrix[0][i - 1]
    }
    for i := 1; i < n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j] = min(f[i - 1][j], f[i - 1][j - 1],f[i - 1][j + 1]) + matrix[i][j - 1]
        }
    }
    ans := 0x3f3f3f3f
    for i := 1; i <= m; i ++ {
        ans = min(f[n - 1][i], ans)
    }
    return ans
}
```

