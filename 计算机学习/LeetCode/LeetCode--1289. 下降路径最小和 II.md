[1289. 下降路径最小和 II](https://leetcode.cn/problems/minimum-falling-path-sum-ii/)

> 给你一个 `n x n` 整数矩阵 `grid` ，请你返回 **非零偏移下降路径** 数字和的最小值。
>
> **非零偏移下降路径** 定义为：从 `grid` 数组中的每一行选择一个数字，且按顺序选出来的数字中，相邻数字不在原数组的同一列。

---

遍历每一个元素的状态的时候，还需要遍历前一行的所有元素的状态：

```go
func minFallingPathSum(grid [][]int) int {
    n := len(grid)
    ans := 0x3f3f3f3f
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, n)
    }
    for i := 0; i < n; i ++ {
        f[0][i] = grid[0][i]
    }
    for i := 1; i < n; i ++ {
        for j := 0; j < n; j ++ {
            mincost := 0x3f3f3f3f
            for k := range grid[i - 1] {
                if k != j {
                    mincost = min(mincost, grid[i][j] + f[i - 1][k])
                }
            }
            f[i][j] = mincost
        }
    }
    for i := 0; i < n; i ++ {
        ans = min(ans, f[n - 1][i])
    }
    return ans
}
```

方法二，也可以选择记录当前的最小的总和以及第二小的总和，这样保证在遇到相同列的元素的时候，可以及时换用另一个，就可以优化时间和空间复杂度：

```go
func minFallingPathSum(grid [][]int) int {
    n := len(grid)
    FirMin, SecMin := 0, 0
    FirIdx := -1
    for i := 0; i < n; i ++ {
        CurFirMin, CurSecMin := 0x3f3f3f3f, 0x3f3f3f3f
        CurFirIdx := -1
        for j := 0; j < n; j ++ {
            Cur := grid[i][j]
            if j != FirIdx {
                Cur += FirMin
            } else {
                Cur += SecMin
            }
            if Cur < CurFirMin {
                CurSecMin, CurFirMin = CurFirMin, Cur
                CurFirIdx = j
            } else if Cur < CurSecMin {
                CurSecMin = Cur
            }
        }
        FirMin, SecMin = CurFirMin, CurSecMin
        FirIdx = CurFirIdx
    }
    return FirMin
}
```

