[1838. 最高频元素的频数](https://leetcode.cn/problems/frequency-of-the-most-frequent-element/)

> 元素的 **频数** 是该元素在一个数组中出现的次数。
>
> 给你一个整数数组 `nums` 和一个整数 `k` 。在一步操作中，你可以选择 `nums` 的一个下标，并将该下标对应元素的值增加 `1` 。
>
> 执行最多 `k` 次操作后，返回数组中最高频元素的 **最大可能频数** *。*

---

依旧排序，这道题只需要排序，然后计算差值即可，如果超出 k，就变更左边界。

```go
func maxFrequency(nums []int, k int) int {
    sort.Ints(nums)
    l := 0
    ans := 1
    cnt := 0
    for r := 1; r < len(nums); r ++ {
        cnt += (r - l) * (nums[r] - nums[r - 1])
        for cnt > k {
            cnt -= (nums[r] - nums[l])
            l ++
        }
        ans = max(ans, r - l + 1)
    }
    return ans
}
```

