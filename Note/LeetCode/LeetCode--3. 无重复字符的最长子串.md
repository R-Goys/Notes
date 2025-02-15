[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 最长子串的长度。

---

easy题目，哈希表+滑动窗口思想

```go
func lengthOfLongestSubstring(s string) int {
    mp := make(map[byte]int)
    i := 0
    MaxLen := 0
    for j := 0; j < len(s); j ++ {
        mp[s[j]]++
        for mp[s[j]] > 1 {
            mp[s[i]]--
            i ++
        }
        MaxLen = max(MaxLen, j - i + 1)
    }
    return MaxLen
}
```

