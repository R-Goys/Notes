[2915. 和为目标值的最长子序列的长度](https://leetcode.cn/problems/length-of-the-longest-subsequence-that-sums-to-target/)

> 给你一个下标从 **0** 开始的整数数组 `nums` 和一个整数 `target` 。
>
> 返回和为 `target` 的 `nums` 子序列中，子序列 **长度的最大值** 。如果不存在和为 `target` 的子序列，返回 `-1` 。
>
> **子序列** 指的是从原数组中删除一些或者不删除任何元素后，剩余元素保持原来的顺序构成的数组。

---

背包，没啥好说的，每次取最大值。

```go
func lengthOfLongestSubsequence(nums []int, target int) int {
    n := len(nums)
    f := make([]int, target + 1)
    for i := 0; i < n; i ++ {
        for j := target; j >= nums[i]; j -- {
            if j - nums[i] == 0 || f[j - nums[i]] > 0 {
                f[j] = max(f[j], f[j - nums[i]] + 1)
            }
        }
    }
    if f[target] == 0 {
        return -1
    }
    return f[target]
}
```

