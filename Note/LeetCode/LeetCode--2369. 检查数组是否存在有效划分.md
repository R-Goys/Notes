[2369. 检查数组是否存在有效划分](https://leetcode.cn/problems/check-if-there-is-a-valid-partition-for-the-array/)

> 给你一个下标从 **0** 开始的整数数组 `nums` ，你必须将数组划分为一个或多个 **连续** 子数组。
>
> 如果获得的这些子数组中每个都能满足下述条件 **之一** ，则可以称其为数组的一种 **有效** 划分：
>
> 1. 子数组 **恰** 由 `2` 个相等元素组成，例如，子数组 `[2,2]` 。
> 2. 子数组 **恰** 由 `3` 个相等元素组成，例如，子数组 `[4,4,4]` 。
> 3. 子数组 **恰** 由 `3` 个连续递增元素组成，并且相邻元素之间的差值为 `1` 。例如，子数组 `[3,4,5]` ，但是子数组 `[1,3,5]` 不符合要求。
>
> 如果数组 **至少** 存在一种有效划分，返回 `true` ，否则，返回 `false` 。

---

一次遍历就行，然后根据给定的条件进行状态转移。

```go
func validPartition(nums []int) bool {
    n := len(nums)
    f := make([]bool, n + 1)

    f[0] = true
    for i := 0; i < n; i ++ {
            if i > 0 && nums[i - 1] == nums[i] {
                f[i + 1] = f[i - 1] || f[i + 1]
            }
            if i > 1 && nums[i - 2] == nums[i - 1] && nums[i - 1] == nums[i] {
                f[i + 1] = f[i - 2] || f[i + 1]
            }
            if i > 1 && nums[i - 2] + 2 == nums[i - 1] + 1 && nums[i - 1] + 1 == nums[i] {
                f[i + 1] = f[i - 2] || f[i + 1]
            }
    }
    return f[n]
}
```

