[1493. 删掉一个元素以后全为 1 的最长子数组](https://leetcode.cn/problems/longest-subarray-of-1s-after-deleting-one-element/)

> 给你一个二进制数组 `nums` ，你需要从中删掉一个元素。
>
> 请你在删掉元素的结果数组中，返回最长的且只包含 1 的非空子数组的长度。
>
> 如果不存在这样的子数组，请返回 0 。

---

依旧滑窗，不过不需要哈希表了

```go
func longestSubarray(nums []int) int {
    cnt := 0
    l := 0
    ans := 0
    for r, x := range nums {
        if x == 0 {
            cnt ++
        }
        for cnt > 1 {
            if nums[l] == 0 {
                cnt --
            }
            l ++
        }
        // 这里不需要 + 1 是因为我们需要返回删除元素之后的长度
        ans = max(ans, r - l)
    }
    return ans
}
```

