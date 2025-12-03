[713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/)

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你返回子数组内所有元素的乘积严格小于 `k` 的连续子数组的数目。

---

前缀和思路，然后滑窗即可，当大于等于给定的 k 时，移动左窗口

```go
func numSubarrayProductLessThanK(nums []int, k int) int {
    if k <= 1 {
        return 0
    }
    ans := 0
    l := 0
    cur := 1
    for r, x := range nums {
        cur *= x
        for cur >= k {
            cur /= nums[l]
            l ++
        }
        ans += r - l + 1
    }
    return ans
}
```

