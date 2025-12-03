[1248. 统计「优美子数组」](https://leetcode.cn/problems/count-number-of-nice-subarrays/)

> 给你一个整数数组 `nums` 和一个整数 `k`。如果某个连续子数组中恰好有 `k` 个奇数数字，我们就认为这个子数组是「**优美子数组**」。
>
> 请返回这个数组中 **「优美子数组」** 的数目。

---

前缀和，我们只需要知道在前缀有 sum 个奇数的时候，对应多少个数组，然后直接根据前缀和算出来。

```go
func numberOfSubarrays(nums []int, k int) int {
    mp := make(map[int]int)
    mp[0] = 1
    sum := 0
    ans := 0
    for _, x := range nums {
        sum += x % 2
        mp[sum] ++
        if sum >= k {
            ans += mp[sum - k]
        }
    }
    return ans
}
```

