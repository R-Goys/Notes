[498. 对角线遍历](https://leetcode.cn/problems/diagonal-traverse/)

> 给你一个大小为 `m x n` 的矩阵 `mat` ，请以对角线遍历的顺序，用一个数组返回这个矩阵中的所有元素。

没啥算法，就是模拟，虽然不难，但是有几点刚刚做的时候卡住了，一是在处理下半部分的对角线的遍历，我最开始没有考虑地全面，这里的x和y的坐标变换没想出来怎么变，

然后这里怎么处理？可以用min和max函数取边界最小值，这样就可以正常变换了，条件也不需要额外去判断，就出现了比较优雅的代码：

```go
func findDiagonalOrder(mat [][]int) []int {
    n, m := len(mat), len(mat[0])
    ans := make([]int, 0)
    for p := 0; p < m + n - 1; p ++{
        if p % 2 == 0 {
            x := min(p, n - 1)
            y := max(p - n + 1, 0)
            for x >= 0 && y < m {
                ans = append(ans, mat[x][y])
                x --
                y ++
            }
        } else {
            x := max(p - m + 1, 0)
            y := min(p, m - 1)
            for x < n && y >= 0 {
                ans = append(ans, mat[x][y])
                x ++
                y --
            }
        }
    }
    return ans
}
```

