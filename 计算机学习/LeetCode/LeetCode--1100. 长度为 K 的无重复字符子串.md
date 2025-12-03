[1100. 长度为 K 的无重复字符子串](https://leetcode.cn/problems/find-k-length-substrings-with-no-repeated-characters/)

> 给你一个字符串 `S`，找出所有长度为 `K` 且不含重复字符的子串，请你返回全部满足要求的子串的 **数目**。

---

和前面的题都是一个套路，遍历，判断，因为是固定长度，所以这样很简单。

```go
func numKLenSubstrNoRepeats(s string, k int) int {
    mp := make([]int, 26)
    sum := 0
    ans := 0
    for i, x := range s {
        if mp[x - 'a'] == 1 {
            sum ++
        }
        mp[x - 'a'] ++
        if i < k - 1 {
            continue
        }
        if sum == 0 {
            ans ++
        }
        mp[s[i - k + 1] - 'a'] --
        if mp[s[i - k + 1] - 'a'] == 1 {
            sum --
        }
    }
    return ans
}
```

