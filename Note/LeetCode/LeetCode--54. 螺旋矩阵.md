[54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

> 给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

----

利用left，right，top，bottom四个遍历维护遍历的边界：

```go
func spiralOrder(matrix [][]int) []int {
    var ans []int
    left, right, top, bottom := 0, len(matrix[0]) - 1, 0, len(matrix) - 1

    for left <= right && top <= bottom {
        for y := left; y <= right; y ++ {
            ans = append(ans, matrix[top][y])
        }
        top ++
        for x := top; x <= bottom; x ++ {
            ans = append(ans, matrix[x][right])
        }
        right --
        if left <= right && top <= bottom{   
            for y := right; y >= left; y -- {
                ans = append(ans, matrix[bottom][y])
            }
            bottom --
            for x := bottom; x >= top; x -- {
                ans = append(ans, matrix[x][left])
            }
            left++
        }
    }
    return ans
}
```

