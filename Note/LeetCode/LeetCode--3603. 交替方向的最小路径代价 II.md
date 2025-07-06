[3603. 交替方向的最小路径代价 II](https://leetcode.cn/problems/minimum-cost-path-with-alternating-directions-ii/)

> 给你两个整数 `m` 和 `n`，分别表示网格的行数和列数。
>
> 进入单元格 `(i, j)` 的成本定义为 `(i + 1) * (j + 1)`。
>
> 另外给你一个二维整数数组 `waitCost`，其中 `waitCost[i][j]` 定义了在该单元格 **等待** 的成本。
>
> 你从第 1 秒开始在单元格 `(0, 0)`。
>
> 每一步，你都遵循交替模式：
>
> - 在 **奇数秒** ，你必须向 **右** 或向 **下** 移动到 **相邻** 的单元格，并支付其进入成本。
> - 在 **偶数秒** ，你必须原地 **等待** ，并支付 `waitCost[i][j]`。
>
> 返回到达 `(m - 1, n - 1)` 所需的 **最小** 总成本。

---

很简单的 dp，结果测试样例卡 0x3f3f3f3f ，周赛还过不去

```go
func minCost(m int, n int, waitCost [][]int) int64 {
    f := make([][]int64, m + 1)
    for i := range f {
        f[i] = make([]int64, n + 1)
        for j := range f[i] {
            f[i][j] = math.MaxInt
        }
    }
    f[0][1] = -int64(waitCost[0][0])
    for i := 1; i <= m; i ++ {
        for j := 1; j <= n; j ++ {
            f[i][j] = min(f[i - 1][j], f[i][j - 1]) + int64(waitCost[i - 1][j - 1] + i * j)
        }
    }
    return f[m][n] - int64(waitCost[m-1][n-1])
}
```

