[1208. 尽可能使字符串相等](https://leetcode.cn/problems/get-equal-substrings-within-budget/)

> 给你两个长度相同的字符串，`s` 和 `t`。
>
> 将 `s` 中的第 `i` 个字符变到 `t` 中的第 `i` 个字符需要 `|s[i] - t[i]|` 的开销（开销可能为 0），也就是两个字符的 ASCII 码值的差的绝对值。
>
> 用于变更字符串的最大预算是 `maxCost`。在转化字符串时，总开销应当小于等于该预算，这也意味着字符串的转化可能是不完全的。
>
> 如果你可以将 `s` 的子字符串转化为它在 `t` 中对应的子字符串，则返回可以转化的最大长度。
>
> 如果 `s` 中没有子字符串可以转化成 `t` 中对应的子字符串，则返回 `0`。

---

还是滑窗，这里可以不需要额外的数组，但是为了方便就写了

```go
func equalSubstring(s string, t string, maxCost int) int {
    n := len(s)
    cost := make([]int, n)
    for i := range s {
        cost[i] = abs(int(s[i]) - int(t[i]))
    }
    l := 0
    curCost := 0
    ans := 0
    for r, x := range cost {
        curCost += x
        for curCost > maxCost {
            curCost -= cost[l]
            l ++
        }
        ans = max(ans, r - l + 1)
    }
    return ans
}


func abs(x int) int {
    if x > 0 {
        return x
    }
    return -x
}
```

