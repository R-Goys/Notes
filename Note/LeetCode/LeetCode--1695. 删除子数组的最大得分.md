[1695. 删除子数组的最大得分](https://leetcode.cn/problems/maximum-erasure-value/)

> 给你一个正整数数组 `nums` ，请你从中删除一个含有 **若干不同元素** 的子数组**。**删除子数组的 **得分** 就是子数组各元素之 **和** 。
>
> 返回 **只删除一个** 子数组可获得的 **最大得分** *。*
>
> 如果数组 `b` 是数组 `a` 的一个连续子序列，即如果它等于 `a[l],a[l+1],...,a[r]` ，那么它就是 `a` 的一个子数组。

---

滑动窗口，还是一样的模板

```go
func maximumUniqueSubarray(nums []int) int {
    mp := make(map[int]int)
    ans := 0
    sum := 0
    l := 0
    for _, x := range nums {
        mp[x] ++
        sum += x
        for mp[x] > 1 {
            mp[nums[l]] --
            sum -= nums[l]
            l ++
        }
        ans = max(ans, sum)
    }
    return ans
}
```

