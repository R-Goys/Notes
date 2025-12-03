[1458. 两个子序列的最大点积](https://leetcode.cn/problems/max-dot-product-of-two-subsequences/)

> 给你两个数组 `nums1` 和 `nums2` 。
>
> 请你返回 `nums1` 和 `nums2` 中两个长度相同的 **非空** 子序列的最大点积。
>
> 数组的非空子序列是通过删除原数组中某些元素（可能一个也不删除）后剩余数字组成的序列，但不能改变数字间相对顺序。比方说，`[2,3,5]` 是 `[1,2,3,4,5]` 的一个子序列而 `[1,5,3]` 不是。

---

大概可能理解线性 dp 是想要干啥了，对于这道题，我们需要求最大点积，和求最长公共子序列非常类似，但是这道题要求我们求得每个元素和每个元素的积的和最大，那么一样的，我们也可以进行类似的 dp 。

两层遍历，计算当前两个元素的积，先令我们的 f[i\][j] 等于这个积，因为前面可能为负数，可以舍去，然后将直接根据之前的状态计算就行了。

```go
func maxDotProduct(nums1 []int, nums2 []int) int {
    n, m := len(nums1), len(nums2)
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m)
    }
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            f[i][j] = nums1[i] * nums2[j]
            if i > 0 {
                f[i][j] = max(f[i - 1][j], f[i][j])
            }
            if j > 0 {
                f[i][j] = max(f[i][j - 1], f[i][j])
            }
            if i > 0 && j > 0 {
                f[i][j] = max(f[i][j], f[i - 1][j - 1] + nums1[i] * nums2[j])
            }
        }
    }
    return f[n - 1][m - 1]
}
```

