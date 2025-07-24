[456. 132 模式](https://leetcode.cn/problems/132-pattern/)

> 给你一个整数数组 `nums` ，数组中共有 `n` 个整数。**132 模式的子序列** 由三个整数 `nums[i]`、`nums[j]` 和 `nums[k]` 组成，并同时满足：`i < j < k` 和 `nums[i] < nums[k] < nums[j]` 。
>
> 如果 `nums` 中存在 **132 模式的子序列** ，返回 `true` ；否则，返回 `false` 。

---

倒序遍历的单调栈：

```go
func find132pattern(nums []int) bool {
    pre := math.MinInt
    st := make([]int, 0)
    for i := len(nums) - 1; i >= 0; i -- {
        if nums[i] < pre {
            return true
        }
        for len(st) > 0 && st[len(st) - 1] < nums[i] {
            pre = st[len(st) - 1]
            st = st[:len(st) - 1]
        }
        st = append(st, nums[i])
    }
    return false
}
```

