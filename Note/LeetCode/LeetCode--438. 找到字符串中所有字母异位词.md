[438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

> 给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

---

上一道题[567. 字符串的排列](https://leetcode.cn/problems/permutation-in-string/)的变式，多了一个统计下标的步骤。

```go
func findAnagrams(s string, p string) []int {
    ans := make([]int, 0)
    mp := make(map[byte]int)
    for i, _ := range p {
        mp[p[i]] ++
    }
    k := len(p)
    l := 0
    src := make(map[byte]int)
    for r, _ := range s {
        src[s[r]] ++
        for mp[s[r]] < src[s[r]] {
            src[s[l]] --
            l ++
        }
        if r - l < k - 1 {
            continue
        }
        ans = append(ans, l)
        src[s[l]] --
        l ++
    }
    return ans
}
```

