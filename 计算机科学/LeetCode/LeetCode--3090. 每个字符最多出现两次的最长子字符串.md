[3090. 每个字符最多出现两次的最长子字符串](https://leetcode.cn/problems/maximum-length-substring-with-two-occurrences/)

> 给你一个字符串 `s` ，请找出满足每个字符最多出现两次的最长子字符串，并返回该子字符串的 **最大** 长度。

---

和上一题一样，哈希表 + 滑动窗口

```go
func maximumLengthSubstring(s string) int {
    l := 0
    mp := make(map[byte]int)
    ans := 0
    for r := range s{
        mp[s[r]] ++ 
        for mp[s[r]] > 2 {
            mp[s[l]] --
            l ++
        }
        ans = max(ans, r - l + 1)
    }  
    return ans
}
```

