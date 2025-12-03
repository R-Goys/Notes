[1818. 绝对差值和](https://leetcode.cn/problems/minimum-absolute-sum-difference/)

> 给你两个正整数数组 `nums1` 和 `nums2` ，数组的长度都是 `n` 。
>
> 数组 `nums1` 和 `nums2` 的 **绝对差值和** 定义为所有 `|nums1[i] - nums2[i]|`（`0 <= i < n`）的 **总和**（**下标从 0 开始**）。
>
> 你可以选用 `nums1` 中的 **任意一个** 元素来替换 `nums1` 中的 **至多** 一个元素，以 **最小化** 绝对差值和。
>
> 在替换数组 `nums1` 中最多一个元素 **之后** ，返回最小绝对差值和。因为答案可能很大，所以需要对 `109 + 7` **取余** 后返回。
>
> `|x|` 定义为：
>
> - 如果 `x >= 0` ，值为 `x` ，或者
> - 如果 `x <= 0` ，值为 `-x`

---

排序 nums1 数组，然后 nums1 对 nums2 中的每个元素进行二分查找，找到差值符合条件的进行替换。

```go
func minAbsoluteSumDiff(nums1 []int, nums2 []int) int {
    src := append([]int{}, nums1...)
    sort.Ints(src)
    sub := 0
    mod := 1000000007
    for i, x := range nums2 {
        idx := sort.SearchInts(src, x)
        origin := abs(nums1[i] - nums2[i])
        se1 := abs(src[min(len(nums1) - 1, idx)] - nums2[i])
        se2 := abs(src[max(0, idx - 1)] - nums2[i])
        sub = max(origin - se1, origin - se2, sub)
    }
    ans := -sub
    for i := range nums1 {
        ans = (ans + abs(nums1[i] - nums2[i])) % mod
    }
    return ans
}

func abs(x int) int {
    if x > 0 {
        return x
    }
    return -x
}
```

