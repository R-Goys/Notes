[132. 分割回文串 II](https://leetcode.cn/problems/palindrome-partitioning-ii/)

> 给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是回文串。
>
> 返回符合要求的 **最少分割次数** 。

---

先状态转移，获取哪些区间 [i, j] 满足回文，到这一步比较简单，第二步比较难思考，需要从状态中提取出最少分割子串的个数来知道分割次数，我们需要倒序枚举，此时也需要在进行一次 dp：

```go
func minCut(s string) int {
    n := len(s)
    f := make([][]bool, n)
    for i := range f {
        f[i] = make([]bool, n)
    }
    for i := n - 1; i >= 0; i -- {
        for j := i; j < n; j ++ {
            if s[i] == s[j] && j - i <= 2 {
                f[i][j] = true
            } else if s[i] == s[j] {
                f[i][j] = f[i + 1][j - 1]
            }
        }
    }

    dp := make([]int, n + 1)
    for i := range dp {
        dp[i] = n - i - 1
    }
    for i := n - 2; i >= 0; i-- {
        // 如果当前字符到最后一个字符都是回文串，说明 ok。
        if f[i][n - 1] {
            dp[i] = 0
        } else {
            // 枚举当前字符之后的位置
            for j := i + 1; j < n; j++ {
                if f[i][j - 1] {
                    dp[i] = min(dp[i], dp[j] + 1)
                }
            }
        }
    }
    
    return dp[0]
}
```

