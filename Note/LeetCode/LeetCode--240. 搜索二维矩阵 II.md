[240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

> 编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：
>
> - 每行的元素从左到右升序排列。
> - 每列的元素从上到下升序排列。

---

### 看成二叉搜索树

```go
func searchMatrix(matrix [][]int, target int) bool {
    n, m := len(matrix), len(matrix[0])
    i, j := 0, m - 1
    for j >= 0 && i < n {
        if matrix[i][j] < target {
            i ++
        } else if matrix[i][j] > target {
            j --
        } else {
            return true
        }
    }
    return false
}
```

