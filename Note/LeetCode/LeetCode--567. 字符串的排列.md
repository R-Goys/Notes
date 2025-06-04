[567. 字符串的排列](https://leetcode.cn/problems/permutation-in-string/)

> 给你两个字符串 `s1` 和 `s2` ，写一个函数来判断 `s2` 是否包含 `s1` 的 排列。如果是，返回 `true` ；否则，返回 `false` 。
>
> 换句话说，`s1` 的排列之一是 `s2` 的 **子串** 。

---

滑窗，因为 s1 的排列是子串，所以相当于定窗滑动窗口，不过有一点点不一样，就是针对于每个新遍历的元素都需要判断它是否存在于 s1 中，s2 是否超出了 s1 所需要的字符串的范围

```go
func checkInclusion(s1 string, s2 string) bool {
    mp := make(map[byte]int)
    for i, _ := range s1 {
        mp[s1[i]] ++
    }
    k := len(s1)
    l := 0
    src := make(map[byte]int)
    for r, _ := range s2 {
        src[s2[r]] ++
        for mp[s2[r]] < src[s2[r]] {
            src[s2[l]] --
            l ++
        }
        if r - l < k - 1 {
            continue
        }
        return true
    }
    return false
}
```

