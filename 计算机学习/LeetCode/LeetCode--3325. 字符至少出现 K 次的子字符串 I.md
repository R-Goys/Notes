[3325. 字符至少出现 K 次的子字符串 I](https://leetcode.cn/problems/count-substrings-with-k-frequency-characters-i/)

> 给你一个字符串 `s` 和一个整数 `k`，在 `s` 的所有子字符串中，请你统计并返回 **至少有一个** 字符 **至少出现** `k` 次的子字符串总数。
>
> **子字符串** 是字符串中的一个连续、 **非空** 的字符序列。

---

哈希表存储每个字符出现的次数，如果当前滑动窗口的右边界对应在哈希表中的值刚好到达了 k，则说明我们当前滑动窗口对应的子字符串符合条件，可以加入结果，与此同时，由于接下来需要移动左边界，所以我们需要把当前左边界对应的所有的符合条件的字符串加入答案，即 `len(s) - r`

```go
func numberOfSubstrings(s string, k int) int {
    mp := ['z' + 1]int{}
    l := 0
    ans := 0
    for r, x := range s {
        mp[x] ++
        for mp[x] == k {
            mp[s[l]] --
            l ++
            ans += len(s) - r
        }
    }
    return ans
}
```

