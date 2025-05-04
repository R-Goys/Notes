[2321. 拼接数组的最大分数](https://leetcode.cn/problems/maximum-score-of-spliced-array/)

> 给你两个下标从 **0** 开始的整数数组 `nums1` 和 `nums2` ，长度都是 `n` 。
>
> 你可以选择两个整数 `left` 和 `right` ，其中 `0 <= left <= right < n` ，接着 **交换** 两个子数组 `nums1[left...right]` 和 `nums2[left...right]` 。
>
> - 例如，设 `nums1 = [1,2,3,4,5]` 和 `nums2 = [11,12,13,14,15]` ，整数选择 `left = 1` 和 `right = 2`，那么 `nums1` 会变为 `[1,***12\*,\*13\***,4,5]` 而 `nums2` 会变为 `[11,***2,3***,14,15]` 。
>
> 你可以选择执行上述操作 **一次** 或不执行任何操作。
>
> 数组的 **分数** 取 `sum(nums1)` 和 `sum(nums2)` 中的最大值，其中 `sum(arr)` 是数组 `arr` 中所有元素之和。
>
> 返回 **可能的最大分数** 。
>
> **子数组** 是数组中连续的一个元素序列。`arr[left...right]` 表示子数组包含 `nums` 中下标 `left` 和 `right` 之间的元素**（含** 下标 `left` 和 `right` 对应元素**）**。

---

很简单，直接计算两个数组的差值，然后算出差值的最大子数组和以及最小子数组和，随后与我们的两个数组和计算即可：

```go
func maximumsSplicedArray(nums1 []int, nums2 []int) int {
    n := len(nums1)
    MinVal, MaxVal, preMin, preMax := 0, 0, 0, 0
    for i := 0; i < n; i ++ {
        preMax = max(nums1[i] - nums2[i], preMax + nums1[i] - nums2[i])
        MaxVal = max(preMax, MaxVal)
        preMin = min(nums1[i] - nums2[i], preMin + nums1[i] - nums2[i])
        MinVal = min(preMin, MinVal)
    }
    sum1, sum2 := 0, 0
    for i := 0; i < n; i ++ {
        sum1 += nums1[i]
        sum2 += nums2[i]
    }
    return max(sum1 - MinVal, sum2 + MaxVal)
}
```

最后的一步，我们需要交换数组以获得最大值，则交换之后，我们的 nums1 可以得到的最大值则是 nums 减去差值为负数时候的最大值，而 nums2 这是加上差值为整数时候的最大值。