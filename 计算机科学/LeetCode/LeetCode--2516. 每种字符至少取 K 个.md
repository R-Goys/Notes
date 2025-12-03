[2516. 每种字符至少取 K 个](https://leetcode.cn/problems/take-k-of-each-character-from-left-and-right/)

> 给你一个由字符 `'a'`、`'b'`、`'c'` 组成的字符串 `s` 和一个非负整数 `k` 。每分钟，你可以选择取走 `s` **最左侧** 还是 **最右侧** 的那个字符。
>
> 你必须取走每种字符 **至少** `k` 个，返回需要的 **最少** 分钟数；如果无法取到，则返回 `-1` 。

---

滑窗，这道题有点坑，必须要三个字符全部包含，还需要给存储数据的哈希表给这三个字符初始化为 0 才行。

```go
func takeCharacters(s string, k int) int {
    mp := make(map[rune]int)
    mp['a'] = 0
    mp['b'] = 0
    mp['c'] = 0
    for _, x := range s {
        mp[x] ++
    }
    for i, x := range mp {
        if x < k {
            return -1
        }
        mp[i] -= k
    }
    l := 0
    ans := 0
    for r, x := range s {
        mp[x] --
        for mp[x] < 0 {
            mp[rune(s[l])] ++
            l ++
        }
        ans = max(r - l + 1, ans)
    }
    return len(s) - ans
}
```

