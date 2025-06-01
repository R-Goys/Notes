[3567. 子矩阵的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-sliding-submatrix/)

> 给你一个 `m x n` 的整数矩阵 `grid` 和一个整数 `k`。
>
> 对于矩阵 `grid` 中的每个连续的 `k x k` **子矩阵**，计算其中任意两个 **不同**值 之间的 **最小绝对差** 。
>
> 返回一个大小为 `(m - k + 1) x (n - k + 1)` 的二维数组 `ans`，其中 `ans[i][j]` 表示以 `grid` 中坐标 `(i, j)` 为左上角的子矩阵的最小绝对差。
>
> **注意**：如果子矩阵中的所有元素都相同，则答案为 0。
>
> 子矩阵 `(x1, y1, x2, y2)` 是一个由选择矩阵中所有满足 `x1 <= x <= x2` 且 `y1 <= y <= y2` 的单元格 `matrix[x][y]` 组成的矩阵。

---

周赛题目，依旧 dfs

```go
func minAbsDiff(grid [][]int, k int) [][]int {
    n, m := len(grid), len(grid[0])
    res := make([][]int, n - k + 1)
    for i := 0 ; i < n - k + 1; i ++ {
        res[i] = make([]int, m - k + 1)
    }
    
    var count func(Ist, Jst int) int 
    count = func(Ist, Jst int) int {
        tmp := make([]int, 0)
        for i := Ist; i < Ist + k; i ++ {
            for j := Jst; j < Jst + k; j ++ {
                tmp = append(tmp, grid[i][j])
            }
        }
        sort.Ints(tmp)
        length := len(tmp)
        ans := 0x3f3f3f3f
        for i := 1; i < length; i ++ {
            if tmp[i] == tmp[i - 1] {
                continue
            }
            ans = min(ans, tmp[i] - tmp[i - 1])
        }
        if ans == 0x3f3f3f3f {
            return 0
        }
        return ans
    }

    for i := 0; i < n - k + 1; i ++ {
        for j := 0; j < m - k + 1; j ++ {
            res[i][j] = count(i, j)
        }
    }
    return res
}
```

