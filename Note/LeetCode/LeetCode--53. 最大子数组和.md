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

