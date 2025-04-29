[265. 粉刷房子 II](https://leetcode.cn/problems/paint-house-ii/)

> 假如有一排房子共有 `n` 幢，每个房子可以被粉刷成 `k` 种颜色中的一种。房子粉刷成不同颜色的花费成本也是不同的。你需要粉刷所有的房子并且使其相邻的两个房子颜色不能相同。
>
> 每个房子粉刷成不同颜色的花费以一个 `n x k` 的矩阵表示。
>
> - 例如，`costs[0][0]` 表示第 `0` 幢房子粉刷成 `0` 号颜色的成本；`costs[1][2]` 表示第 `1` 幢房子粉刷成 `2` 号颜色的成本，以此类推。
>
> 返回 *粉刷完所有房子的最低成本* 。

---

时间 O（NK^2） 解法，和粉刷房子 I 类似。

```go
func minCostII(costs [][]int) int {
    n := len(costs)
    m := len(costs[0])
    f := make([][]int, n + 1)
    ans := 1000000
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m)
    }

    for i := 1; i <= n; i ++ {
        for color := 0; color < m; color ++ {
            f[i][color] = 114514
            for k := 0; k < m; k ++ {
                if k != color {
                    f[i][color] = min(f[i][color], f[i - 1][k] + costs[i - 1][color])
                }
            }
        }
    }
    for i := 0; i < m; i ++ {
        ans = min(ans, f[n][i])
    }
    return ans
}
```

当然，这里可以使用滚动数组来优化，但是我们的重点放在第二种解法上面

我们的解法 2 需要实现时间复杂度 O（nk） 的算法，我们可以在处理第二层循环的时候将这个嵌套循环分为两部分，第一部分一层 for 循环找到前面房子开销的最小值和第二小值，这里是因为我们在计算当前状态的时候，如果发现最小值的颜色和当前要涂的颜色相同，就只能去用第二小的值。于是，我们就可以写出来了：

```go
func minCostII(costs [][]int) int {
    n := len(costs)
    m := len(costs[0])
    f := make([][]int, n + 1)
    ans := 1000000
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m)
    }
    for i := 1; i <= n; i ++ {
        minCost1 := -1 
        minColor1 := -1 
        minCost2 := -1 
        minColor2 := -1

        for color := 0; color < m; color++ {
            currentCost := f[i-1][color]
            if minColor1 == -1 || currentCost < minCost1 {
                minCost2 = minCost1
                minColor2 = minColor1
                minCost1 = currentCost 
                minColor1 = color
            } else if minColor2 == -1 || currentCost < minCost2 {
                minCost2 = currentCost
                minColor2 = color
            }
        }
        
        for color := 0; color < m; color++ {
            if color == minColor1 {
                f[i][color] = f[i - 1][minColor2] + costs[i - 1][color]
            } else {
                f[i][color] = f[i - 1][minColor1] + costs[i - 1][color]
            }
        }
    }
    for i := 0; i < m; i ++ {
        ans = min(ans, f[n][i])
    }
    return ans
}
```

