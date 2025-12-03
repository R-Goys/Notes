[2841. 几乎唯一子数组的最大和](https://leetcode.cn/problems/maximum-sum-of-almost-unique-subarray/)

> 给你一个整数数组 `nums` 和两个正整数 `m` 和 `k` 。
>
> 请你返回 `nums` 中长度为 `k` 的 **几乎唯一** 子数组的 **最大和** ，如果不存在几乎唯一子数组，请你返回 `0` 。
>
> 如果 `nums` 的一个子数组有至少 `m` 个互不相同的元素，我们称它是 **几乎唯一** 子数组。
>
> 子数组指的是一个数组中一段连续 **非空** 的元素序列。

----

同样的套路，我这里还需要使用哈希表来记录当前子数组中的不同元素的数量：

```go
func maxSum(nums []int, m int, k int) int64 {
    mp := make(map[int]int, 0)
    sum, ans := 0, 0
    cnt := 0
    for i, x := range nums {
        sum += x
        if mp[x] == 0 {
            cnt ++
        }
        mp[x] ++
        if i < k - 1 {
            continue
        }
        if cnt >= m {
            ans = max(ans, sum)
        }
        sum -= nums[i - k + 1]
        if mp[nums[i - k + 1]] --; mp[nums[i - k + 1]] == 0 {
            cnt --
        }
        
    }
    return int64(ans)
}
```

