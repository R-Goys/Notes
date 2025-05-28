[643. 子数组最大平均数 I](https://leetcode.cn/problems/maximum-average-subarray-i/)

> 给你一个由 `n` 个元素组成的整数数组 `nums` 和一个整数 `k` 。
>
> 请你找出平均数最大且 **长度为 `k`** 的连续子数组，并输出该最大平均数。
>
> 任何误差小于 `10-5` 的答案都将被视为正确答案。

---

滑动窗口，秒杀

```go
func findMaxAverage(nums []int, k int) float64 {
    var sum float64 = 0.0
    var ans float64 = - 114524
    n := len(nums)
    l, r := 0, 0
    for r < n {
        for r - l < k && r < n {
            sum += float64(nums[r])
            r ++
        }
        if r - l == k && l < n {
            ans = max(ans, sum / float64(k))
            sum -= float64(nums[l])
            l ++
        }
    }
    return ans
}
```

