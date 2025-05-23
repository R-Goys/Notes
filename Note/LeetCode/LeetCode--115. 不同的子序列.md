[115. 不同的子序列](https://leetcode.cn/problems/distinct-subsequences/)

> 给你两个字符串 `s` 和 `t` ，统计并返回在 `s` 的 **子序列** 中 `t` 出现的个数。
>
> 测试用例保证结果在 32 位有符号整数范围内。

---

动态规划，由于是两个字符串相比较，跟绝大多数的字符串比较的思路都类似，我们需要思考应该如何才能够正确表示在一个字符串中能够存在多少个另一个字符串，很明显，我们应该开辟二维数组，使用`f[i][j]`来表示在**字符串1**前`i`个字符中，存在的**字符串2**中的前`j`个字符的数量，如何表示？

如果当我们的i和j是相同的字符，则可以表示为：**字符串1**前`i - 1`个字符包含的**字符串2**前`j`个字符的数量加上**字符串1**前`i - 1`个字符中包含的**字符串2**前`j - 1`字符的数量，然后，对于初始状态，对于所有的空字串""，**字符串1**包含它的数量应该为1，所以代码如下：

```go
func numDistinct(s string, t string) int {
    n, m := len(s), len(t)
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
        f[i][0] = 1
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if s[i - 1] == t[j - 1] {
                f[i][j] = f[i - 1][j - 1] + f[i - 1][j]
            } else {
                f[i][j] = f[i - 1][j]
            }
        }
    }
    return f[n][m]
}
```

二刷：

刚做不久又忘了，哎，多练吧：

```go
func numDistinct(s string, t string) int {
    n, m := len(s), len(t)
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
        f[i][0] = 1
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if s[i - 1] == t[j - 1] {
                f[i][j] = f[i - 1][j] + f[i - 1][j - 1]
            } else {
                f[i][j] = f[i - 1][j]
            }
        }
    }
    return f[n][m]
}
```

