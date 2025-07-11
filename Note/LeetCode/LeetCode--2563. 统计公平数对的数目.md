[2563. 统计公平数对的数目](https://leetcode.cn/problems/count-the-number-of-fair-pairs/)

> 给你一个下标从 **0** 开始、长度为 `n` 的整数数组 `nums` ，和两个整数 `lower` 和 `upper` ，返回 **公平数对的数目** 。
>
> 如果 `(i, j)` 数对满足以下情况，则认为它是一个 **公平数对** ：
>
> - `0 <= i < j < n`，且
> - `lower <= nums[i] + nums[j] <= upper`

---

排序 + 二分，没啥好说的。

```go
func countFairPairs(nums []int, lower int, upper int) int64 {
    sort.Ints(nums)
    var ans int64 = 0
    for i, x := range nums {
        l := sort.SearchInts(nums[:i], lower - x)
        r := sort.SearchInts(nums[:i], upper - x + 1)
        ans += int64(r - l)
    }
    return ans
}
```

