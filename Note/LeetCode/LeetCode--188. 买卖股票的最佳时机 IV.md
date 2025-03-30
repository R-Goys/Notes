[188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)

> 给你一个整数数组 `prices` 和一个整数 `k` ，其中 `prices[i]` 是某支给定的股票在第 `i` 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。
>
> **注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

---

没写过这个题，雀食🐂🍺，思路是这样的，buy表示第i次买入股票时候的最大值，sell表示第i次卖出股票的最大值，

初始情况下，我们将他们都初始化为很小的数值，为防止溢出，这里要除以2.

同时要记得初始化第一天buy的值，然后开始我们的dp。

第一层循环是遍历每一天，将每一天的状态都更新一下，第二层循环，我们需要更新每一次买入卖出的状态。

```go
func maxProfit(k int, prices []int) int {
    n := len(prices)
    if n == 0 {
        return 0
    }

    k = min(k, n / 2)
    buy := make([]int, k + 1)
    sell := make([]int, k + 1)
    buy[0] = -prices[0]
    for i := 1; i <= k; i ++ {
        buy[i] = math.MinInt64 / 2
        sell[i] = math.MinInt64 / 2
    }

    for i := 1; i < n; i ++ {
        //可以选择是否把今天当作第一天买入，保证最优性。
        buy[0] = max(buy[0], sell[0] - prices[i])
        for j := 1; j <= k ; j ++ {
            //表示当天根据之前保存卖出的状态买入，还是之前保存的买入。
            buy[j] = max(buy[j], sell[j] - prices[i])
            //表示卖出上次出售的，以及之前保存的卖出状态。
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

由于我们是根据之前的状态来进行买入卖出，所以可以保证不会出现第二天买入在第一天卖出这种情况。