[1234. 替换子串得到平衡字符串](https://leetcode.cn/problems/replace-the-substring-for-balanced-string/)

> 有一个只含有 `'Q', 'W', 'E', 'R'` 四种字符，且长度为 `n` 的字符串。
>
> 假如在该字符串中，这四个字符都恰好出现 `n/4` 次，那么它就是一个「平衡字符串」。
>
>  
>
> 给你一个这样的字符串 `s`，请通过「替换一个子串」的方式，使原字符串 `s` 变成一个「平衡字符串」。
>
> 你可以用和「待替换子串」长度相同的 **任何** 其他字符串来完成替换。
>
> 请返回待替换子串的最小可能长度。
>
> 如果原字符串自身就是一个平衡字符串，则返回 `0`。

---

先统计字符串里面所有字符的数量，然后滑窗表示替换的字符串，如果滑窗之外的所有字符都满足小于 len / 4 的条件，那么一定可以替换，因为我们只能替换字符串，也就是说可以增加某一个字符串的数量，但是不能减少，所以此时一定正确

```go
func balancedString(s string) int {
    m := len(s) / 4
    mp := make(map[rune]int)
    for _, x := range s {
        mp[x] ++
    }
    if mp['Q'] == m && mp['W'] == m && mp['E'] == m && mp['R'] == m {
        return 0
    }
    ans := len(s)
    l := 0
    for r, x := range s {
        mp[x] --
        for mp['Q'] <= m && mp['W'] <= m && mp['E'] <= m && mp['R'] <= m {
            ans = min(ans, r - l + 1)
            mp[rune(s[l])] ++
            l ++
        }
    }
    return ans
}
```

