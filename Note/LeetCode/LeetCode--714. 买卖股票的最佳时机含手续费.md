[714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

> 给定一个整数数组 `prices`，其中 `prices[i]`表示第 `i` 天的股票价格 ；整数 `fee` 代表了交易股票的手续费用。
>
> 你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。
>
> 返回获得利润的最大值。
>
> **注意：**这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

---

easy题目加个小费，还是easy

```go
func maxProfit(prices []int, fee int) int {
    n := len(prices)
    f := make([][2]int, n)
    f[0][1] = -prices[0]
    for i := 1; i < n; i ++ {
        f[i][0] = max(f[i - 1][1] - fee + prices[i], f[i - 1][0])
        f[i][1] = max(f[i - 1][0] - prices[i], f[i - 1][1])
    }
    return max(f[n - 1][0], f[n - 1][1])
}
```

