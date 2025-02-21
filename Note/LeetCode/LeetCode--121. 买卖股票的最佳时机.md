[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

> 给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。
>
> 你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。
>
> 返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

----

统计前缀的价格最小值，一次遍历，然后减去即可得到最大利润

```go
func maxProfit(prices []int) int {
    MinPrice := 10000
    ans := 0
    for i := 0; i < len(prices); i++ {
        MinPrice = min(MinPrice, prices[i])
        ans = max(ans, prices[i] - MinPrice)
    }
    return ans
}
```

