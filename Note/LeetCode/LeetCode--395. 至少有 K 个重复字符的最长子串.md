[395. 至少有 K 个重复字符的最长子串](https://leetcode.cn/problems/longest-substring-with-at-least-k-repeating-characters/)

> 给你一个字符串 `s` 和一个整数 `k` ，请你找出 `s` 中的最长子串， 要求该子串中的每一字符出现次数都不少于 `k` 。返回这一子串的长度。
>
> 如果不存在这样的子字符串，则返回 0。

---

原本准备用滑动窗口，后面发现分治更简单

```go
func longestSubstring(s string, k int) int {
    n := len(s)
    if n < k {
        return 0
    }
    var record [26]int
    for _, c := range s {
        record[c - 'a'] ++
    }
    for key, val := range record {
        if 0 < val && val < k {
            ans := 0
            for _, son := range strings.Split(s, string(byte(key) + 'a')) {
                ans = max(ans, longestSubstring(son, k))
            }
            return ans
        }
    }
    return n
}
```

利用所有出现次数小于k的字符将字符串分割，同时递归到下一层查找，简单暴力。

这里不得不提一下官方题解那个分治和滑动窗口的题解真的挺抽象，不太清晰感觉。