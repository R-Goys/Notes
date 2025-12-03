[2606. 找到最大开销的子字符串](https://leetcode.cn/problems/find-the-substring-with-maximum-cost/)

> 给你一个字符串 `s` ，一个字符 **互不相同** 的字符串 `chars` 和一个长度与 `chars` 相同的整数数组 `vals` 。
>
> **子字符串的开销** 是一个子字符串中所有字符对应价值之和。空字符串的开销是 `0` 。
>
> **字符的价值** 定义如下：
>
> - 如果字符不在字符串 `chars` 中，那么它的价值是它在字母表中的位置（下标从 1 开始）。
>   - 比方说，`'a'` 的价值为 `1` ，`'b'` 的价值为 `2` ，以此类推，`'z'` 的价值为 `26` 。
> - 否则，如果这个字符在 `chars` 中的位置为 `i` ，那么它的价值就是 `vals[i]` 。
>
> 请你返回字符串 `s` 的所有子字符串中的最大开销。

---

和求最大子数组和一样，就是多了个处理键值对。

```go
func maximumCostSubstring(s string, chars string, vals []int) int {
    mp := make(map[rune]int, 0)
    pre, Max := 0, 0
    for i, k := range chars {
        mp[k] = vals[i]
    }
    for _, k := range s {
        if _, ok := mp[k]; !ok {
            mp[k] = int(k - 'a') + 1
        }
    }
    for _, v := range s {
        pre = max(mp[v], pre + mp[v])
        Max = max(pre, Max)
    }
    return Max
}
```

