[2302. 统计得分小于 K 的子数组数目](https://leetcode.cn/problems/count-subarrays-with-score-less-than-k/)

> 一个数组的 **分数** 定义为数组之和 **乘以** 数组的长度。
>
> - 比方说，`[1, 2, 3, 4, 5]` 的分数为 `(1 + 2 + 3 + 4 + 5) * 5 = 75` 。
>
> 给你一个正整数数组 `nums` 和一个整数 `k` ，请你返回 `nums` 中分数 **严格小于** `k` 的 **非空整数子数组数目**。
>
> **子数组** 是数组中的一个连续元素序列。

---

纸老虎，一开始看是 hard 还吓一跳，结果随便秒杀。依旧前缀和思想。

```go
func countSubarrays(nums []int, k int64) int64 {
    var sum int64 = 0
    l := 0
    var ans int64 = 0
    for r, x := range nums {
        sum += int64(x)
        for sum * int64(r - l + 1) >= k {
            sum -= int64(nums[l])
            l ++
        }
        ans += int64(r - l + 1)
    }
    return ans
}
```

