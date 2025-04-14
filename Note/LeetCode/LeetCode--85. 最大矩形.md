[85. 最大矩形](https://leetcode.cn/problems/maximal-rectangle/)

> 给定一个仅包含 `0` 和 `1` 、大小为 `rows x cols` 的二维二进制矩阵，找出只包含 `1` 的最大矩形，并返回其面积。

---

前缀和扫描

计算每一行连续的1的长度，然后拿到遍历相同列的行的长度的最小值，计算出结果。

```go
func maximalRectangle(matrix [][]byte) int {
    n, m := len(matrix), len(matrix[0])
    max := 0
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m)
    }

    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            if matrix[i][j] == '1' {
                f[i][j] = 1
                if j > 0 {
                    f[i][j] += f[i][j - 1]
                }
            }
            for k, min := i, f[i][j]; k >= 0 && min > 0; k -- {
                if min > f[k][j] {
                    min = f[k][j]
                }
                if area := int(min) * (i - k + 1); max < area {
                    max = area
                }
            }
        }
    }
    return max
}
```

