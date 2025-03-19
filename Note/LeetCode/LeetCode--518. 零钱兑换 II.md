[518. 零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)

> 给你一个整数数组 `coins` 表示不同面额的硬币，另给一个整数 `amount` 表示总金额。
>
> 请你计算并返回可以凑成总金额的硬币组合数。如果任何硬币组合都无法凑出总金额，返回 `0` 。
>
> 假设每一种面额的硬币有无限个。 
>
> 题目数据保证结果符合 32 位带符号整数。

---

超高时间，空间复杂度纯享版：

```go
func change(amount int, coins []int) int {
    n := len(coins)
    f := make([][]int, amount + 1)
    for i := 0; i <= amount; i ++ {
        f[i] = make([]int, n + 1)
    }
    for i := 0; i <= n; i ++ {
        f[0][i] = 1
    }

    for i := 1; i <= amount; i ++ {
        for j := 1; j <= n; j ++ {
            f[i][j] = f[i][j - 1]
            for k := 1; k*coins[j - 1] <= i; k ++ {
                f[i][j] += f[i - k * coins[j - 1]][j - 1]
            }
        }
    }
    return f[amount][n]
}
```

优化：

```go
func change(amount int, coins []int) int {
    n := len(coins)
    f := make([]int, amount + 1)
    f[0] = 1

    for j := 1; j <= n; j ++ {
        for i := coins[j - 1]; i <= amount; i ++ {
            f[i] += f[i - coins[j - 1]]
        }
    }
    return f[amount]
}
```

