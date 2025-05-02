[53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

>给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
>**子数组**是数组中的一个连续部分。

---

前缀和思想：

```go
func maxSubArray(nums []int) int {
    n := len(nums)
    pre, MaxVal := nums[0], nums[0]
    for i := 1; i < n; i ++ {
        pre = max(nums[i], pre + nums[i])
        MaxVal = max(pre, MaxVal)
    }
    return MaxVal
}
```

二刷，这里的意思是如果前缀的贡献小于零，那么我们就需要舍去前面的前缀，从当前数字重新计算。

```go
func maxSubArray(nums []int) int {
    n := len(nums)
    pre, MaxVal := nums[0], nums[0]
    for i := 1; i < n; i ++ {
        pre = max(nums[i], pre + nums[i])
        MaxVal = max(MaxVal, pre)
    }
    return MaxVal
}
```

另一种做法是保存前缀的最小值，然后使用当前的前缀去减去前缀最小值。

```go
func maxSubArray(nums []int) int {
    pre, Max, Min := 0, -111111, 0
    for i := 0; i < len(nums); i ++ {
        Min = min(Min, pre)
        pre += nums[i]
        Max = max(pre - Min, Max)
    }
    return Max
}
```

