[2743. 计算没有重复字符的子字符串数量](https://leetcode.cn/problems/count-substrings-without-repeating-character/)

> 给定你一个只包含小写英文字母的字符串 `s` 。如果一个子字符串不包含任何字符至少出现两次（换句话说，它不包含重复字符），则称其为 **特殊** 子字符串。你的任务是计算 **特殊** 子字符串的数量。例如，在字符串 `"pop"` 中，子串 `"po"` 是一个特殊子字符串，然而 `"pop"` 不是 **特殊** 子字符串（因为 `'p'` 出现了两次）。
>
> 返回 **特殊** 子字符串的数量。
>
> **子字符串** 是指字符串中连续的字符序列。例如，`"abc"` 是 `"abcd"` 的一个子字符串，但 `"acd"` 不是。

---

[LCP 68. 美观的花束](https://leetcode.cn/problems/1GxJYY/)和这道题一模一样

```go
func numberOfSpecialSubstrings(s string) int {
    mp := ['z' + 1]int{}
    l := 0
    ans := 0
    for r, x := range s {
        mp[x] ++
        for mp[x] > 1 {
            mp[s[l]] --
            l ++
        }
        ans += r - l + 1
    }
    return ans
}
```

