[2958. 最多 K 个重复元素的最长子数组](https://leetcode.cn/problems/length-of-longest-subarray-with-at-most-k-frequency/)

> 给你一个整数数组 `nums` 和一个整数 `k` 。
>
> 一个元素 `x` 在数组中的 **频率** 指的是它在数组中的出现次数。
>
> 如果一个数组中所有元素的频率都 **小于等于** `k` ，那么我们称这个数组是 **好** 数组。
>
> 请你返回 `nums` 中 **最长好** 子数组的长度。
>
> **子数组** 指的是一个数组中一段连续非空的元素序列。

---

露头就秒，还是太模板了

```go
func maxSubarrayLength(nums []int, k int) int {
    mp := make(map[int]int)
    l := 0
    ans := 0
    for r, x := range nums {
        mp[x] ++
        for mp[x] > k {
            mp[nums[l]] -- 
            l ++
        }
        ans = max(ans, r - l + 1)
    }
    return ans
}
```

