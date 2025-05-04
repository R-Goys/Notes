[560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。
>
> 子数组是数组中元素的连续非空序列

---

前缀和，哈希表存前缀个数，遇见匹配上的ans直接加上哈希表中当前元素的个数。

```go
func subarraySum(nums []int, k int) int {
    n := len(nums)
    m := make(map[int]int)
    ans := 0
    prev := 0
    m[0] = 1
    
    for i := 0; i < n; i ++ {
        prev += nums[i]
        tmp := prev - k
        if _, ok := m[tmp]; ok {
            ans += m[tmp]
        }
        m[prev] ++
    }
    return ans
}
```

