[122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

> 给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。
>
> 在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。
>
> 返回 *你能获得的 **最大** 利润* 。

---

## dp

为方便理解，贴一个没有优化的版本：

d\[i][0]指的是当天不持有股票的最大利润，d\[i][1]表示当天持有股票时拥有的最大利润

```go
func maxProfit(prices []int) int {
    n := len(prices)
    dp := make([][2]int, n)
    dp[0][1] = -prices[0]
    for i := 1; i < n; i ++ {
        dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i])
        dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i])
    }
    return dp[n - 1][0]
}
```

由于只需要前一个状态，可以优化为一维数组：

```go
func maxProfit(prices []int) int {
    n := len(prices)
    dp := make([]int, 2)
    dp[1] = -prices[0]
    for i := 1; i < n; i ++ {
        dp[0] = max(dp[0], dp[1] + prices[i])
        dp[1] = max(dp[1], dp[0] - prices[i])
    }
    return dp[0]
}
```

