[1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

---

>给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。
>
>一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
>- 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。
>
>两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

## DP

rt，经典动态规划，逐层遍历字符串，i表示text1的遍历到的元素下标，j表示text2遍历到的元素下标。

那么每遍历到一个元素，若两个字符串当前遍历的元素不相同，则`f[i][j] = max(f[i][j - 1], f[i - 1][j])`，也就是之前两个状态的最长子序列长度取最大值，如果相同，那么就是`f[i][j] = f[i - 1][j - 1] + 1`，即在之前的状态上 + 1，就得到了当前的最长公共子序列长度，代码如下：

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    n := len(text1)
    m := len(text2)
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j] = max(f[i - 1][j], f[i][j - 1])
            if text1[i - 1] == text2[j - 1] {
                f[i][j] = max(f[i][j], f[i - 1][j - 1] + 1)
            }
        }
    }
    return f[n][m]
}
```

---

