[48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

> 给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。
>
> 你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

---

## 找规律

由于两个两个交换的话，会导致处理很麻烦，但是如果在矩阵中，一次性交换四个元素的位置，处理起来就很方便：

```go
func rotate(matrix [][]int)  {
    n := len(matrix)
    for i := 0; i < n / 2; i ++ {
        for j := 0; j < (n + 1) / 2; j ++ {
            matrix[i][j], matrix[j][n - i - 1], matrix[n - i - 1][n - j - 1], matrix[n - j - 1][i] =  matrix[n - j - 1][i], matrix[i][j], matrix[j][n - i - 1], matrix[n - i - 1][n - j - 1]
        }
    }
}

```

## 两次翻转

先以X轴上下反转，再y = -x这条线对角线反转，这次可以得到我们的旋转图像

```go
func rotate(matrix [][]int)  {
    n := len(matrix)
    for i := 0; i < n / 2 ; i ++ {
        for j := 0; j < n; j ++ {
            matrix[i][j], matrix[n - i - 1][j] = matrix[n - i - 1][j], matrix[i][j]
        }
    }

    for i := 0; i < n; i ++ {
        for j := 0; j < i; j ++ {
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
        }
    }
}
```

