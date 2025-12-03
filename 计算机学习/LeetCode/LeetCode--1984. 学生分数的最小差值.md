[1984. 学生分数的最小差值](https://leetcode.cn/problems/minimum-difference-between-highest-and-lowest-of-k-scores/)

> 给你一个 **下标从 0 开始** 的整数数组 `nums` ，其中 `nums[i]` 表示第 `i` 名学生的分数。另给你一个整数 `k` 。
>
> 从数组中选出任意 `k` 名学生的分数，使这 `k` 个分数间 **最高分** 和 **最低分** 的 **差值** 达到 **最小化** 。
>
> 返回可能的 **最小差值** 。

---

看了半天挺疑惑的，结果是让我排序😓

```go
func minimumDifference(nums []int, k int) int {
    sort.Ints(nums)
    ans := 0x3f3f3f3f
    for i, x := range nums {
        if i < k - 1 {
            continue
        }
        ans = min(x - nums[i - k + 1], ans)
    }
    return ans
}
```

