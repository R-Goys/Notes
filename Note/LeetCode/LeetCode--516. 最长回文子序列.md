[516. 最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence/)

>给你一个字符串 `s` ，找出其中最长的回文子序列，并返回该序列的长度。
>
>子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列。

---

乍一看还以为是leetcode5，一顿操作猛如虎，结果写错了。

本体是求子序列，所以不能求连续的，但是子序列怎么求？很简单，我们只需要将状态转移的条件变化一下即可，在计算长度的时候，如果当前的子序列的左右两边相等，直接正常计算即可，无需管中间如何。如果不相等，从之前的状态继承最长子序列的长度。

```go
func longestPalindromeSubseq(s string) int {
    n := len(s)
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, n)
        f[i][i] = 1
    }
    for i := n - 1; i >= 0; i -- {
        for j := i + 1; j < n; j ++ {
            if s[i] == s[j] {
                f[i][j] = f[i + 1][j - 1] + 2
            } else {
                f[i][j] = max(f[i + 1][j], f[i][j - 1])
            }
        }
    }
    return f[0][n - 1]
}
```

