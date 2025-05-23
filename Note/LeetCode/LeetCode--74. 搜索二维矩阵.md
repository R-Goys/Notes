[74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)

> 给你一个满足下述两条属性的 `m x n` 整数矩阵：
>
> - 每行中的整数从左到右按非严格递增顺序排列。
> - 每行的第一个整数大于前一行的最后一个整数。
>
> 给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

---

利用取模将矩阵展开成一维，随后进行二分搜索：

```go
func searchMatrix(matrix [][]int, target int) bool {
    n, m := len(matrix), len(matrix[0])
    if target < matrix[0][0] || target > matrix[n - 1][m - 1] {
        return false
    }
    l, r := 0, n * m
    for l <= r {
        mid := (l + r) / 2
        if matrix[mid / m][mid % m] < target {
            l = mid + 1
        } else if matrix[mid / m][mid % m] > target {
            r = mid - 1
        } else {
            return true
        }
    }
    return false
}
```

看成二叉搜索树，直接开搜：

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

