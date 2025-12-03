[3576. 数组元素相等转换](https://leetcode.cn/problems/transform-array-to-all-equal-elements/)

> 给你一个大小为 `n` 的整数数组 `nums`，其中只包含 `1` 和 `-1`，以及一个整数 `k`。
>
> 你可以最多进行 `k` 次以下操作：
>
> - 选择一个下标 `i`（`0 <= i < n - 1`），然后将 `nums[i]` 和 `nums[i + 1]` 同时 **乘以** `-1`。
>
> **注意：**你可以在 **不同** 的操作中多次选择相同的下标 `i`。
>
> 如果在最多 `k` 次操作后可以使数组的所有元素相等，则返回 `true`；否则，返回 `false`。

---

分为两种情况，都变成 1 和都变成 -1。

```go
func canMakeEqual(nums []int, k int) bool {
    n := len(nums)
    for target := -1; target <= 1; target += 2 {
        m := k
        tmp := make([]int, n)
        for i := range nums {
            tmp[i] = nums[i]
        }
        
        for i := range tmp {
            if tmp[i] == target && i == n - 1 {
                break
            }
            if tmp[i] == target {
                if m == 0 {
                    break
                }
                tmp[i] *= -1
                tmp[i + 1] *= -1
                m --
            }
            if i == n - 1 && tmp[i] == target * -1 {
                return true
            }
        }
    }
    return false
}
```

