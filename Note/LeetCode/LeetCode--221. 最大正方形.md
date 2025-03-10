[221. 最大正方形](https://leetcode.cn/problems/maximal-square/)

> 在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。

---

## dp

一开始没想出来dp，看了题目提示词试着做了一下，还真就过了。

这个dp应该算比较简单的吧，只不过没想出来这道题是dp，哈哈

最主要的状态转移方程思路就是将`f[i - 1][j]`和`f[i][j - 1]` 以及`f[i - 1][j - 1]`找到，然后根据他们的最小值+1即可，因为正方形的每个部分都满足这个条件，容易推出反过来，也是满足的。

```go
func maximalSquare(matrix [][]byte) int {
    n := len(matrix)
    m := len(matrix[0])
    f := make([][]int, n + 1)
    ans := 0
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
        f[i][0] = 0
    }
    for i := 0; i <= m; i ++ {
        f[0][i] = 0
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if matrix[i - 1][j - 1] == '1' {
                f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1, f[i - 1][j - 1] + 1)
                ans = max(ans, f[i][j])
            }
        }
    }
    return ans*ans
}
```

