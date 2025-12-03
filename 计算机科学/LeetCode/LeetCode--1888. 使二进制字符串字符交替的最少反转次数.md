[1888. 使二进制字符串字符交替的最少反转次数](https://leetcode.cn/problems/minimum-number-of-flips-to-make-the-binary-string-alternating/)

> 给你一个二进制字符串 `s` 。你可以按任意顺序执行以下两种操作任意次：
>
> - **类型 1 ：删除** 字符串 `s` 的第一个字符并将它 **添加** 到字符串结尾。
> - **类型 2 ：选择** 字符串 `s` 中任意一个字符并将该字符 **反转** ，也就是如果值为 `'0'` ，则反转得到 `'1'` ，反之亦然。
>
> 请你返回使 `s` 变成 **交替** 字符串的前提下， **类型 2** 的 **最少** 操作次数 。
>
> 我们称一个字符串是 **交替** 的，需要满足任意相邻字符都不同。
>
> - 比方说，字符串 `"010"` 和 `"1010"` 都是交替的，但是字符串 `"0100"` 不是。

---

滑动窗口意味着我们进行操作一，这里并不消耗次数，而操作二意味着我们需要统计当前窗口有多少不符合我们期望的字符

```go
func minFlips(s string) int {
    n := len(s)
    target := "01"
    cnt := 0
    for i := 0; i < n; i ++ {
        if s[i] != target[i % 2] {
            cnt ++
        }
    }
    
    ans := min(cnt, n - cnt)
    for i := 0; i < n; i ++ {
        if s[i] != target[i % 2] {
            cnt --
        }
        if s[i] != target[(i + n) % 2] {
            cnt ++
        }
        ans = min(ans, cnt, n - cnt)
    }
    return ans
}
```

