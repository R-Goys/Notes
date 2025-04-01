[59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

>给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

---

这道题相比于螺旋矩阵比较特殊，不会出现狭长路径的情况，所以不需要做特殊的边界处理

```go
func generateMatrix(n int) [][]int {
    ans := make([][]int, n)
    for i := 0; i < n; i ++ {
        ans[i] = make([]int, n)
    }
    idx := 0
    left, right, top, bottom := 0, n - 1, 0, n - 1
    for left <= right && top <= bottom {
        for y := left; y <= right; y ++ {
            idx ++
            ans[top][y] = idx
        }
        top++
        for x := top; x <= bottom; x ++ {
            idx ++
            ans[x][right] = idx
        }
        right --
        for y := right; y >= left; y -- {
            idx ++
            ans[bottom][y] = idx
        }
        bottom --
        for x := bottom; x >= top; x -- {
            idx ++
            ans[x][left] = idx
        }
        left ++
    }
    return ans
}
```

二刷

```go
func generateMatrix(n int) [][]int {
    left, right, top, base := 0, n - 1, 0, n - 1
    i, j := 0, -1
    idx := 1
    ans := make([][]int, n)
    for k := 0; k < n; k ++ {
        ans[k] = make([]int, n)
    }
    for left <= right {      
        for j < right {
           j ++
           ans[i][j] = idx
           idx ++
        }
        top ++
        for i < base {
            i ++
            ans[i][j] = idx
            idx ++
        }
        right--
        for j > left {
            j --
            ans[i][j] = idx
            idx ++
        }
        base --
        for i > top {
            i --
            ans[i][j] = idx
            idx ++
        }
        left ++
    }
    return ans
}
```

