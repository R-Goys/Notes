[2730. 找到最长的半重复子字符串](https://leetcode.cn/problems/find-the-longest-semi-repetitive-substring/)

> 给你一个下标从 **0** 开始的字符串 `s` ，这个字符串只包含 `0` 到 `9` 的数字字符。
>
> 如果一个字符串 `t` 中至多有一对相邻字符是相等的，那么称这个字符串 `t` 是 **半重复的** 。例如，`"0010"` 、`"002020"` 、`"0123"` 、`"2002"` 和 `"54944"` 是半重复字符串，而 `"00101022"` （相邻的相同数字对是 00 和 22）和 `"1101234883"` （相邻的相同数字对是 11 和 88）不是半重复字符串。
>
> 请你返回 `s` 中最长 **半重复** 子字符串 的长度。

---

简单的滑动窗口，这里不能用哈希表来判断，而是一位一位去判断，不过核心思路都是一致的

```go
func longestSemiRepetitiveSubstring(s string) int {
    ans := 1
    l := 0
    cnt := 0
    for r := 1; r < len(s); r ++ {
        if s[r] == s[r - 1] {
            cnt ++
        }
        for cnt > 1 {
            if s[l] == s[l + 1] {
                cnt --
            }
            l ++
        }
        ans = max(r - l + 1, ans)
    }
    return ans
}
```

