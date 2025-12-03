[1092. 最短公共超序列](https://leetcode.cn/problems/shortest-common-supersequence/)

> 给你两个字符串 `str1` 和 `str2`，返回同时以 `str1` 和 `str2` 作为 **子序列** 的最短字符串。如果答案不止一个，则可以返回满足条件的 **任意一个** 答案。
>
> 如果从字符串 `t` 中删除一些字符（也可能不删除），可以得到字符串 `s` ，那么 `s` 就是 t 的一个子序列。

---

先求最短公共超序列的长度，然后根据长度，倒序遍历去还原最优解，为什么是倒序而不能是正序？举个例子比如 "cab" 和 "abc" ，此时我们不知道应该选择哪一种作为前缀，而倒序则可以做到这一点，能够做出正确的决断。

```go
func shortestCommonSupersequence(str1 string, str2 string) string {
    n, m := len(str1), len(str2)

    f := make([][]int, n + 1)
    // 求最短公共超序列长度：
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m+1)
        f[i][0] = i
    }
    for j := 1; j <= m; j++ {
        f[0][j] = j
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if str1[i - 1] == str2[j - 1] {
                f[i][j] = f[i - 1][j - 1] + 1
            } else {
                f[i][j] = min(f[i - 1][j], f[i][j - 1]) + 1
            }
        }
    }
    // 倒序组合答案
    ans := make([]byte, 0)
    i, j := n - 1, m - 1
    for i >= 0 && j >= 0 {
        if str1[i] == str2[j] {
            ans = append(ans, str1[i])
            i, j = i - 1, j - 1
        } else if f[i + 1][j + 1] == f[i][j + 1] + 1 { // 根据状态转移的中间量来转移
            ans = append(ans, str1[i])
            i --
        } else {
            ans = append(ans, str2[j])
            j --
        }
    }
    slices.Reverse(ans)
    return str1[:i + 1] + str2[:j + 1] + string(ans)
}
```

