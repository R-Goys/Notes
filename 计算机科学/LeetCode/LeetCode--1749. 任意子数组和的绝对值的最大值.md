[1749. 任意子数组和的绝对值的最大值](https://leetcode.cn/problems/maximum-absolute-sum-of-any-subarray/)

> 给你一个整数数组 `nums` 。一个子数组 `[numsl, numsl+1, ..., numsr-1, numsr]` 的 **和的绝对值** 为 `abs(numsl + numsl+1 + ... + numsr-1 + numsr)` 。
>
> 请你找出 `nums` 中 **和的绝对值** 最大的任意子数组（**可能为空**），并返回该 **最大值** 。
>
> `abs(x)` 定义如下：
>
> - 如果 `x` 是负整数，那么 `abs(x) = -x` 。
> - 如果 `x` 是非负整数，那么 `abs(x) = x` 。

---

和之前的最大子数组和一样，只需要多记录一个最小值就行了

```go
func maxAbsoluteSum(nums []int) int {
    MaxVal, MinVal, preMin, preMax := 0, 0, 0, 0
    n := len(nums)
    for i := 0; i < n ; i ++ {
        preMin = min(preMin + nums[i], nums[i])
        preMax = max(preMax + nums[i], nums[i])
        MinVal = min(MinVal, preMin)
        MaxVal = max(preMax, MaxVal)
    }
    return max(- MinVal, MaxVal)
}
```



