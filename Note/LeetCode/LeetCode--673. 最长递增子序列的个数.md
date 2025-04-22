[673. 最长递增子序列的个数](https://leetcode.cn/problems/number-of-longest-increasing-subsequence/)

>给定一个未排序的整数数组 `nums` ， *返回最长递增子序列的个数* 。
>
>**注意** 这个数列必须是 **严格** 递增的。

---

母题是最长递增子序列这道题，倒是思路上却有一些差异，我们这里采取的是for循环遍历的方式，形式类似于冒泡，相当于是先遍历枚举一个子序列的终点，然后枚举一个子序列的起点，当然，这个子序列的起点的最大长度在之前已经被计算过，所以此处直接使用即可。

如果当我们枚举的起点，其数值小于终点的数值，那么进一步判断，如果当前起点的长度 + 1 > 当前终点的长度，则更新终点的长度，以及更新计数，如果相等，那么修改一下计数，表示相同子序列长度的数量+1即可。

```go
func findNumberOfLIS(nums []int) int {
    maxLen := 0
    ans := 0
    n := len(nums)
    f := make([]int, n)
    cnt := make([]int, n)

    for i := 0; i < n; i ++ {
        f[i] = 1
        cnt[i] = 1
        for j := 0; j < i; j ++ {
            if nums[i] > nums[j] {
                if f[j] + 1 > f[i] {
                    f[i] = f[j] + 1
                    cnt[i] = cnt[j]
                } else if f[j] + 1 == f[i] {
                    cnt[i] += cnt[j]
                }
            }
        }
        if f[i] > maxLen {
            maxLen = f[i]
            ans = cnt[i]
        } else if f[i] == maxLen {
            ans += cnt[i]
        }
    }
    return ans
}
```

