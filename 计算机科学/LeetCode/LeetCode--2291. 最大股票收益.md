[2291. 最大股票收益](https://leetcode.cn/problems/maximum-profit-from-trading-stocks/)

> 给你两个下标从 **0** 开始的数组 `present` 和 `future` ，`present[i]` 和 `future[i]` 分别代表第 `i` 支股票现在和将来的价格。每支股票你最多购买 **一次** ，你的预算为 `budget` 。
>
> 求最大的收益。

---

好久没写过这么简单的题了，最近被难题折磨得不成样了，信心都快没了

背包为题，我们将每一支股票的净收益单独拿出来作为 src 数组，然后单纯的背包状态转移

```go
func maximumProfit(present []int, future []int, budget int) int {
    n := len(present)
    profit := make([]int, n)
    for i := 0; i < n; i ++ {
        profit[i] = future[i] - present[i]
    }
    f := make([]int, budget + 1)
    for i := 0; i < n; i ++ {
        for j := budget; j >= present[i]; j -- {
            f[j] = max(f[j], f[j - present[i]] + profit[i])
        }
    }
    return f[budget]
}
```

