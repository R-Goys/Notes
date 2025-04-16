[97. 交错字符串](https://leetcode.cn/problems/interleaving-string/)

> 给定三个字符串 `s1`、`s2`、`s3`，请你帮忙验证 `s3` 是否是由 `s1` 和 `s2` **交错** 组成的。
>
> 两个字符串 `s` 和 `t` **交错** 的定义与过程如下，其中每个字符串都会被分割成若干 **非空** 子字符串：
>
> - `s = s1 + s2 + ... + sn`
> - `t = t1 + t2 + ... + tm`
> - `|n - m| <= 1`
> - **交错** 是 `s1 + t1 + s2 + t2 + s3 + t3 + ...` 或者 `t1 + s1 + t2 + s2 + t3 + s3 + ...`
>
> **注意：**`a + b` 意味着字符串 `a` 和 `b` 连接。

---

双指针没法做，这样的顺序可能是乱的(比如给出的样例)

最好还是用动态规划，可以根据之前的状态进行计算，而不是单纯的一个双指针表示数值

```go
func isInterleave(s1 string, s2 string, s3 string) bool {
    n, m, t := len(s1), len(s2), len(s3)
    if (m + n) != t {
        return false
    }

    f := make([][]bool, n + 1)
    for i := 0; i <=n; i ++ {
        f[i] = make([]bool, m + 1)
    }
    f[0][0] = true

    for i := 0; i <= n; i ++ {
        for j := 0; j <= m; j ++ {
             p := i + j - 1
             if i > 0 {
                f[i][j] = f[i][j] || (f[i - 1][j] && s1[i - 1] == s3[p])
             }
             if j > 0 {
                f[i][j] = f[i][j] || (f[i][j - 1] && s2[j - 1] == s3[p])
             }
        }
    }
    return f[n][m]
}
```

很显然，这里可以使用滚动数组优化，但是懒得写了)