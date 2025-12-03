[3258. 统计满足 K 约束的子字符串数量 I](https://leetcode.cn/problems/count-substrings-that-satisfy-k-constraint-i/)

> 给你一个 **二进制** 字符串 `s` 和一个整数 `k`。
>
> 如果一个 **二进制字符串** 满足以下任一条件，则认为该字符串满足 **k 约束**：
>
> - 字符串中 `0` 的数量最多为 `k`。
> - 字符串中 `1` 的数量最多为 `k`。
>
> 返回一个整数，表示 `s` 的所有满足 **k 约束** 的子字符串的数量。

---

easy 题目，需要两个判断

```go
func countKConstraintSubstrings(s string, k int) int {
    cnt := ['3']int{}
    ans := 0
    l := 0
    for r, x := range s {
        cnt[x] ++
        for cnt['0'] > k && cnt['1'] > k {
            cnt[s[l]] --
            l ++
        }
        ans += r - l + 1
    }
    return ans
}
```

