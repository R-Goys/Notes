[2904. 最短且字典序最小的美丽子字符串](https://leetcode.cn/problems/shortest-and-lexicographically-smallest-beautiful-string/)

> 给你一个二进制字符串 `s` 和一个正整数 `k` 。
>
> 如果 `s` 的某个子字符串中 `1` 的个数恰好等于 `k` ，则称这个子字符串是一个 **美丽子字符串** 。
>
> 令 `len` 等于 **最短** 美丽子字符串的长度。
>
> 返回长度等于 `len` 且字典序 **最小** 的美丽子字符串。如果 `s` 中不含美丽子字符串，则返回一个 **空** 字符串。
>
> 对于相同长度的两个字符串 `a` 和 `b` ，如果在 `a` 和 `b` 出现不同的第一个位置上，`a` 中该位置上的字符严格大于 `b` 中的对应字符，则认为字符串 `a` 字典序 **大于** 字符串 `b` 。
>
> - 例如，`"abcd"` 的字典序大于 `"abcc"` ，因为两个字符串出现不同的第一个位置对应第四个字符，而 `d` 大于 `c` 。

---

随便写个题放松一下，哈希表存储，然后排序，好暴力，不过我暂时只能想到这个方法。

```go
func shortestBeautifulSubstring(s string, k int) string {
    mp := make(map[int][]string)
    oneNum := 0
    l := 0
    idx := 0x3f3f3f3f
    for r, num := range s {
        if num == '1' {
            oneNum ++
        }
        if oneNum < k {
            continue
        }
        for oneNum > k {
            if s[l] == '1' {
                oneNum --
            }
            l ++
        }
        for s[l] == '0' {
            l ++
        }
        idx = min(r - l + 1, idx)
        mp[r - l + 1] = append(mp[r - l + 1], s[l : r + 1])
    }
    if idx >= 0x3f3f3f3f {
        return ""
    }
    strs := mp[idx]
    sort.Strings(strs)
    return strs[0]
}
```



