[1027. 最长等差数列](https://leetcode.cn/problems/longest-arithmetic-subsequence/)

> 给你一个整数数组 `nums`，返回 `nums` 中最长等差子序列的**长度**。
>
> 回想一下，`nums` 的子序列是一个列表 `nums[i1], nums[i2], ..., nums[ik]` ，且 `0 <= i1 < i2 < ... < ik <= nums.length - 1`。并且如果 `seq[i+1] - seq[i]`( `0 <= i < seq.length - 1`) 的值都相同，那么序列 `seq` 是等差的。

---

最开始就是想到的枚举difference，但是感觉时间复杂度都比较高，直接pass了，回头去看题解还真是这么写的。。

```go
func longestArithSeqLength(nums []int) int {
    n := len(nums)
    m := slices.Max(nums)
    ans := 1
    for i := -m; i <= m; i ++ {
        f := make([]int, m + 1)

        for j := 0 ; j < n; j ++ {
            if nums[j] - i >= 0 && nums[j] - i <= m {
                f[nums[j]] = f[nums[j] - i] + 1
                ans = max(ans, f[nums[j]])
            } else {
                f[nums[j]] = 1
            }
        }
    }
    return ans
}
```

另一种方法，对于特别大的数字，时间复杂度会相对比较低，缺点就是空间复杂度很高。

我们会每一位和每一位的difference，然后f\[i][d]表示以第i个数字结尾的difference为d的序列长度。

```go
func longestArithSeqLength(nums []int) int {
    n := len(nums)
    ans := 0
    f := make([][1001]int, n)
    for i := 1; i < n; i ++ {
        for j := i - 1; j >= 0; j -- {
            d := nums[i] - nums[j] + 500
            if f[i][d] == 0 {
                f[i][d] = f[j][d] + 1
                ans = max(ans, f[i][d])
            }
        }
    }
    return ans + 1
}
```

