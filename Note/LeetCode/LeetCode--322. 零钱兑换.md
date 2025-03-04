[322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

> 给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。
>
> 计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。
>
> 你可以认为每种硬币的数量是无限的。

----

典型动态规划，背包问题

```go
func coinChange(coins []int, amount int) int {
    f := make([]int, amount + 1)
    for i := 0; i <= amount; i ++ {
        f[i] = -1
    }
    f[0] = 0
    
    for i := 0; i < len(coins); i ++ {
        for j := 0; j <= amount; j ++ {
            if coins[i] > j || f[j - coins[i]] == -1 {
                continue
            }
            if f[j] != -1 {
                f[j] = min(f[j], f[j - coins[i]] + 1)
            } else {
                f[j] = f[j - coins[i]] + 1
            }
        }
    }
    return f[amount]
}
```

