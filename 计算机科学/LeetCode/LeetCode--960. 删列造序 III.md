[960. 删列造序 III](https://leetcode.cn/problems/delete-columns-to-make-sorted-iii/)

> 给定由 `n` 个小写字母字符串组成的数组 `strs` ，其中每个字符串长度相等。
>
> 选取一个删除索引序列，对于 `strs` 中的每个字符串，删除对应每个索引处的字符。
>
> 比如，有 `strs = ["abcdef","uvwxyz"]` ，删除索引序列 `{0, 2, 3}` ，删除后为 `["bef", "vyz"]` 。
>
> 假设，我们选择了一组删除索引 `answer` ，那么在执行删除操作之后，最终得到的数组的行中的 **每个元素** 都是按**字典序**排列的（即 `(strs[0][0] <= strs[0][1] <= ... <= strs[0][strs[0].length - 1])` 和 `(strs[1][0] <= strs[1][1] <= ... <= strs[1][strs[1].length - 1])` ，依此类推）。
>
> 请返回 *`answer.length` 的最小可能值* 。

---

最长非降子序列问题，但是我们需要提取出需要删除哪一行。

这里我们直接当作 n 个子序列去做，然后遍历这些字符串切片就可以了。

```go
func minDeletionSize(strs []string) int {
    n := len(strs[0])
    f := make([][]bool, n)
    for i := 0; i < n; i++ {
        f[i] = make([]bool, n)
    }

    for i := 0; i < n - 1; i ++ {
        f[i][i] = true
        for j := i + 1; j < n; j ++ {
            f[i][j] = true
            for _, str := range strs {
                if str[i] > str[j] {
                    f[i][j] = false
                    break
                }
            }
        }
    }
    dp := make([]int, n)
    for i := range dp {
        dp[i] = 1
    }
    
    Max := 1
    for i := 1; i < n; i++ {
        for j := i - 1; j >= 0; j -- {
            if f[j][i] {
                dp[i] = max(dp[i], dp[j] + 1)
            }
        }
        Max = max(dp[i], Max)
    }

    return n - Max
}
```

