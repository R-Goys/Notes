[496. 下一个更大元素 I](https://leetcode.cn/problems/next-greater-element-i/)

> `nums1` 中数字 `x` 的 **下一个更大元素** 是指 `x` 在 `nums2` 中对应位置 **右侧** 的 **第一个** 比 `x` 大的元素。
>
> 给你两个 **没有重复元素** 的数组 `nums1` 和 `nums2` ，下标从 **0** 开始计数，其中`nums1` 是 `nums2` 的子集。
>
> 对于每个 `0 <= i < nums1.length` ，找出满足 `nums1[i] == nums2[j]` 的下标 `j` ，并且在 `nums2` 确定 `nums2[j]` 的 **下一个更大元素** 。如果不存在下一个更大元素，那么本次查询的答案是 `-1` 。
>
> 返回一个长度为 `nums1.length` 的数组 `ans` 作为答案，满足 `ans[i]` 是如上所述的 **下一个更大元素** 。

---

依旧单调栈，借助哈希表进行映射

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
    mp := make(map[int]int)
    for i, x := range nums2 {
        mp[x] = i
    }
    stk := make([]int, 0)
    src := make([]int, len(nums2))
    for i, x := range nums2 {
        src[i] = -1
        for len(stk) > 0 && nums2[stk[len(stk) - 1]] < x {
            idx := stk[len(stk) - 1]
            stk = stk[:len(stk) - 1]
            src[idx] = x
        }
        stk = append(stk, i)
    }
    for i := range nums1 {
        nums1[i] = src[mp[nums1[i]]]
    }
    return nums1
}
```

