[583. 两个字符串的删除操作](https://leetcode.cn/problems/delete-operation-for-two-strings/)

> 给定两个单词 `word1` 和 `word2` ，返回使得 `word1` 和 `word2` **相同**所需的**最小步数**。
>
> **每步** 可以删除任意一个字符串中的一个字符。

---

就是求最长公共子序列

```go
func minDistance(word1 string, word2 string) int {
    n, m := len(word1), len(word2)

    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if word1[i - 1] == word2[j - 1] {
                f[i][j] = f[i - 1][j - 1] + 1
            } else {
                f[i][j] = max(f[i - 1][j], f[i][j - 1])
            }
        }
    }
    return m + n - 2 * f[n][m]
}
```

