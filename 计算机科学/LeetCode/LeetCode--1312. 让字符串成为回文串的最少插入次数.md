[1312. 让字符串成为回文串的最少插入次数](https://leetcode.cn/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

>给你一个字符串 `s` ，每一次操作你都可以在字符串的任意位置插入任意字符。
>
>请你返回让 `s` 成为回文串的 **最少操作次数** 。
>
>「回文串」是正读和反读都相同的字符串。

---

翻译过来就是求最长回文子序列的长度，然后使用当前字符串的长度减去最长回文子序列的长度即可。

为什么是这样？我们需要插入字符，使得这个字符串成为最终的回文字符串，比较容易推断出我们最少的操作次数，其实就是在最长回文子序列的基础上进行插入。

```go
func minInsertions(s string) int {
    n := len(s)
    dp := make([][]int, n)
    for i := range dp {
        dp[i] = make([]int, n)
        dp[i][i] = 1
    }

    for length := 2; length <= n; length++ {
        for i := 0; i <= n - length; i++ {
            j := i + length - 1
            if s[i] == s[j] {
                dp[i][j] = dp[i+1][j-1] + 2
            } else {
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
            }
        }
    }
    return n - dp[0][n-1]
}
```

