[2401. 最长优雅子数组](https://leetcode.cn/problems/longest-nice-subarray/)

> 给你一个由 **正** 整数组成的数组 `nums` 。
>
> 如果 `nums` 的子数组中位于 **不同** 位置的每对元素按位 **与（AND）**运算的结果等于 `0` ，则称该子数组为 **优雅** 子数组。
>
> 返回 **最长** 的优雅子数组的长度。
>
> **子数组** 是数组中的一个 **连续** 部分。
>
> **注意：**长度为 `1` 的子数组始终视作优雅子数组。

---

利用位运算的特性进行滑动窗口，按位或运算之后再和该数字进行异或运算就可以恢复原样，由此来进行滑动窗口。

```go
func longestNiceSubarray(nums []int) int {
    l, or := 0, 0
    ans := 0
    for r, x := range nums {
        for or & x > 0 {
            or ^= nums[l]
            l ++
        }
        or |= x
        ans = max(ans, r - l + 1)
    }
    return ans
}
```

