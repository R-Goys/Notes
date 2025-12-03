[2962. 统计最大元素出现至少 K 次的子数组](https://leetcode.cn/problems/count-subarrays-where-max-element-appears-at-least-k-times/)

> 给你一个整数数组 `nums` 和一个 **正整数** `k` 。
>
> 请你统计有多少满足 「 `nums` 中的 **最大** 元素」至少出现 `k` 次的子数组，并返回满足这一条件的子数组的数目。
>
> 子数组是数组中的一个连续元素序列。

---

这里不能完全按照之前的[1358. 包含所有三种字符的子字符串数目](https://leetcode.cn/problems/number-of-substrings-containing-all-three-characters/)的思路来做，我们这里求的是数组最大值在其子数组中至少存在 k 个的子数组个数，换个思路，如果我们当前子数组已经有了 k 个目标值，那么此时它的左边界左移多少位就可以满足条件，换而言之，就是把右边界看作不变量，根据左边界和索引为 0 的差值进行计算，就是当前 [0, r] 中，右边界为 r 的符合条件的子数组的个数。

```go
func countSubarrays(nums []int, k int) int64 {
    maxNum := 0
    for _, x := range nums {
        maxNum = max(maxNum, x)
    }
    cnt := 0
    ans := 0
    l := 0
    for _, x := range nums {
        if x == maxNum {
            cnt ++
        }
        for cnt == k {
            if nums[l] == maxNum {
                cnt --
            }
            l ++
        }
        ans += l
    }
    return int64(ans)
}
```

