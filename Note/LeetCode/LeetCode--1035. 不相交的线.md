[1035. 不相交的线](https://leetcode.cn/problems/uncrossed-lines/)

> 在两条独立的水平线上按给定的顺序写下 `nums1` 和 `nums2` 中的整数。
>
> 现在，可以绘制一些连接两个数字 `nums1[i]` 和 `nums2[j]` 的直线，这些直线需要同时满足：
>
> -  `nums1[i] == nums2[j]`
> - 且绘制的直线不与任何其他连线（非水平线）相交。
>
> 请注意，连线即使在端点也不能相交：每个数字只能属于一条连线。
>
> 以这种方法绘制线条，并返回可以绘制的最大连线数。

这道题就是求最长公共子序列，知道这点直接秒了。

```go
func maxUncrossedLines(nums1 []int, nums2 []int) int {
    n, m := len(nums1), len(nums2)
    f := make([][]int, 2)
    for i := 0; i < 2; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if nums1[i - 1] == nums2[j - 1] {
                f[i % 2][j] = f[(i - 1) % 2][j - 1] + 1
            } else {
                f[i % 2][j] = max(f[(i - 1) % 2][j], f[i % 2][j - 1])
            }
        }
    }
    return f[n % 2][m]
}
```

