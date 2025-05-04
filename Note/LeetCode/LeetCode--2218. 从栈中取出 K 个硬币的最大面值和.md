[2218. 从栈中取出 K 个硬币的最大面值和](https://leetcode.cn/problems/maximum-value-of-k-coins-from-piles/)

> 一张桌子上总共有 `n` 个硬币 **栈** 。每个栈有 **正整数** 个带面值的硬币。
>
> 每一次操作中，你可以从任意一个栈的 **顶部** 取出 1 个硬币，从栈中移除它，并放入你的钱包里。
>
> 给你一个列表 `piles` ，其中 `piles[i]` 是一个整数数组，分别表示第 `i` 个栈里 **从顶到底** 的硬币面值。同时给你一个正整数 `k` ，请你返回在 **恰好** 进行 `k` 次操作的前提下，你钱包里硬币面值之和 **最大为多少** 。

依旧是一种背包问题，但是略有不同，他是通过栈这种形式来取出物品的，但是我们依旧可以通过传统遍历去解决它，但是此时需要记录从前 i 个栈中取 ，取 j 次硬币可以获取的最大值。

```go
func maxValueOfCoins(piles [][]int, k int) int {
    n := len(piles)
    f := make([][]int, n + 1)
    for i := range f {
        f[i] = make([]int, k + 1)
    }
    for i := 0; i < n; i ++ {
        for j := 0; j <= k; j ++ {
            f[i + 1][j] = f[i][j]
            v := 0
            for w := 0; w < min(j, len(piles[i])); w ++ {
                v += piles[i][w]
                f[i + 1][j] = max(f[i + 1][j], f[i][j - w - 1] + v)
            }
        }
    }
    return f[n][k]
}
```

