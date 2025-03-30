[123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

> 给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。
>
> **注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

---

看了这种代码，我只能说牛逼。

```go
func maxProfit(prices []int) int {
    buy1, sell1, buy2, sell2 := -prices[0], 0, -prices[0], 0
    for i := 1; i < len(prices); i ++ {
        buy1 = max(buy1, - prices[i])
        sell1 = max(sell1, buy1 + prices[i])
        buy2 = max(buy2, sell1 - prices[i])
        sell2 = max(sell2, buy2 + prices[i])
    }
    return sell2
}
```

做完买卖股票的最佳时期IV的我回来降维打击了😎

```go
func maxProfit(prices []int) int {
    n := len(prices)
    if n == 0 {
        return 0
    }
    k := 2
    buy := make([]int, k + 1)
    sell := make([]int, k + 1)
    buy[0] = -prices[0]
    for i := 1; i <= k; i ++ {
        buy[i] = math.MinInt64 / 2
        sell[i] = math.MinInt64 / 2
    }

    for i := 1; i < n; i ++ {
        buy[0] = max(buy[0], sell[0] - prices[i])
        for j := 1; j <= k ; j ++ {
            buy[j] = max(buy[j], sell[j] - prices[i])
            sell[j] = max(sell[j], buy[j - 1] + prices[i])
        }
    }
    return max(sell...)
}

func max(a ...int) int {
    res := a[0]
    for _, v := range a[1:] {
        if v > res {
            res = v
        }
    }
    return res
}
```

