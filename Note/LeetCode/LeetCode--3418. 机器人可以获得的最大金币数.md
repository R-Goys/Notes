[3418. 机器人可以获得的最大金币数](https://leetcode.cn/problems/maximum-amount-of-money-robot-can-earn/)

> 给你一个 `m x n` 的网格。一个机器人从网格的左上角 `(0, 0)` 出发，目标是到达网格的右下角 `(m - 1, n - 1)`。在任意时刻，机器人只能向右或向下移动。
>
> 网格中的每个单元格包含一个值 `coins[i][j]`：
>
> - 如果 `coins[i][j] >= 0`，机器人可以获得该单元格的金币。
> - 如果 `coins[i][j] < 0`，机器人会遇到一个强盗，强盗会抢走该单元格数值的 **绝对值** 的金币。
>
> 机器人有一项特殊能力，可以在行程中 **最多感化** 2个单元格的强盗，从而防止这些单元格的金币被抢走。
>
> **注意：**机器人的总金币数可以是负数。
>
> 返回机器人在路径上可以获得的 **最大金币数** 。

---

简单的做法只需要开辟三维数组进行各种状态的转移即可，唯一值得注意的是初始状态需要正确的初始化。

```go
func maximumAmount(coins [][]int) int {
    n, m := len(coins), len(coins[0])
    f := make([][][3]int, n + 1)
    ans := -0x3f3f3f3f
    for i := 0; i <= n; i ++ {
        f[i] = make([][3]int, m + 1)
        for j := 0; j < 3; j ++ {
            f[i][0][j] = -0x3f3f3f3f
        }
    }
    for i := 0; i <= m; i ++ {
        for j := 0; j < 3; j ++ {
            f[0][i][j] = -0x3f3f3f3f
        }
    }
    f[0][1][0] = 0
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j][0] = max(f[i - 1][j][0], f[i][j - 1][0]) + coins[i - 1][j - 1]
            f[i][j][1] = max(f[i - 1][j][1], f[i][j - 1][1]) + coins[i - 1][j - 1]
            f[i][j][2] = max(f[i - 1][j][2], f[i][j - 1][2]) + coins[i - 1][j - 1]

            if coins[i - 1][j - 1] < 0 {
                f[i][j][1] = max(f[i - 1][j][0], f[i][j - 1][0], f[i][j][1])
                f[i][j][2] = max(f[i - 1][j][1], f[i][j - 1][1], f[i][j][2])
            }
        }
    }
    for i := 0; i < 3; i ++ {
        ans = max(f[n][m][i], ans)
    }
    return ans
}
```

