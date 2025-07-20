[1475. 商品折扣后的最终价格](https://leetcode.cn/problems/final-prices-with-a-special-discount-in-a-shop/)

> 给你一个数组 `prices` ，其中 `prices[i]` 是商店里第 `i` 件商品的价格。
>
> 商店里正在进行促销活动，如果你要买第 `i` 件商品，那么你可以得到与 `prices[j]` 相等的折扣，其中 `j` 是满足 `j > i` 且 `prices[j] <= prices[i]` 的 **最小下标** ，如果没有满足条件的 `j` ，你将没有任何折扣。
>
> 请你返回一个数组，数组中第 `i` 个元素是折扣后你购买商品 `i` 最终需要支付的价格。

---

单调栈

```go
func finalPrices(prices []int) []int {
    stk := make([]int, 0)
    for i, x := range prices {
        for len(stk) > 0 && prices[stk[len(stk) - 1]] >= x {
            idx := stk[len(stk) - 1]
            stk = stk[:len(stk) - 1]
            prices[idx] -= x
        }
        stk = append(stk, i)
    }
    return prices
}

```

