[2389. 和有限的最长子序列](https://leetcode.cn/problems/longest-subsequence-with-limited-sum/)

> 给你一个长度为 `n` 的整数数组 `nums` ，和一个长度为 `m` 的整数数组 `queries` 。
>
> 返回一个长度为 `m` 的数组 `answer` ，其中 `answer[i]` 是 `nums` 中 元素之和小于等于 `queries[i]` 的 **子序列** 的 **最大** 长度 。
>
> **子序列** 是由一个数组删除某些元素（也可以不删除）但不改变剩余元素顺序得到的一个数组。

---

前缀和 + 二分

```go
func answerQueries(nums []int, queries []int) []int {
    sort.Ints(nums)
    pref := make([]int, len(nums))
    for i, x := range nums {
        if i == 0 {
            pref[i] = x
            continue
        }
        pref[i] = pref[i - 1] + x
    }
    ans := make([]int, 0)
    for _, x := range queries {
        ans = append(ans, sort.SearchInts(pref, x + 1))
    }
    return ans
}
```

