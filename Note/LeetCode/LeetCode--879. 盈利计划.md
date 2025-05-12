[879. 盈利计划](https://leetcode.cn/problems/profitable-schemes/)

> 集团里有 `n` 名员工，他们可以完成各种各样的工作创造利润。
>
> 第 `i` 种工作会产生 `profit[i]` 的利润，它要求 `group[i]` 名成员共同参与。如果成员参与了其中一项工作，就不能参与另一项工作。
>
> 工作的任何至少产生 `minProfit` 利润的子集称为 **盈利计划** 。并且工作的成员总数最多为 `n` 。
>
> 有多少种计划可以选择？因为答案很大，所以 **返回结果模** `10^9 + 7` **的值**。

---

题目不难，就是读题有点费劲💩

实际上就是一个 01 背包问题，但是多了一维关于利润，实际上状态转移方程还是一样的，一样的地方在于，我们先遍历"组"，再遍历"容量"，最后遍历利润即可，值得提及的就是我们的状态转移计算的式子中，利润的计算为：`max(0, k - profit[i])` 这里的目的是获取工作利润至少为 k 的子集的数量，而不是工作利润为 k 的数量。

这样一来，当我们计算工作利润至少为 k 的子集的数量时，当我们的 profit[i] 大于我们当前计算的利润，此时依旧是从零进行状态转移，这样就确保了正确得到子集数量，这里很妙。

```go
func profitableSchemes(n int, minProfit int, group []int, profit []int) int {
    mod := 1000000000 + 7
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, minProfit + 1)
        f[i][0] = 1
    }
    for i := 0; i < len(group); i ++ {
        for j := n; j >= group[i]; j -- {
            for k := minProfit; k >= 0; k -- {
                f[j][k] = (f[j][k] + f[j - group[i]][max(0, k - profit[i])]) % mod
            }
        }
    }
    return f[n][minProfit]
}
```

