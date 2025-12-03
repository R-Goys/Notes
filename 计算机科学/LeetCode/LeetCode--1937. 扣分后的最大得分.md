[1937. 扣分后的最大得分](https://leetcode.cn/problems/maximum-number-of-points-with-cost/)

> 给你一个 `m x n` 的整数矩阵 `points` （下标从 **0** 开始）。一开始你的得分为 `0` ，你想最大化从矩阵中得到的分数。
>
> 你的得分方式为：**每一行** 中选取一个格子，选中坐标为 `(r, c)` 的格子会给你的总得分 **增加** `points[r][c]` 。
>
> 然而，相邻行之间被选中的格子如果隔得太远，你会失去一些得分。对于相邻行 `r` 和 `r + 1` （其中 `0 <= r < m - 1`），选中坐标为 `(r, c1)` 和 `(r + 1, c2)` 的格子，你的总得分 **减少** `abs(c1 - c2)` 。
>
> 请你返回你能得到的 **最大** 得分。
>
> `abs(x)` 定义为：
>
> - 如果 `x >= 0` ，那么值为 `x` 。
> - 如果 `x < 0` ，那么值为 `-x` 。

---

根据 0 神的思路来推的话，就比较简单了，我们需要知道前面一行的除去当前点的前缀和后缀的最大值，以此来计算：

```go
func maxPoints(points [][]int) int64 {
    ans := 0
    n := len(points[0])
    f := make([][2]int, n)
    sufMax := make([]int, n)
    for i, row := range points {
        if i == 0 {
            for j, v := range row {
                ans = max(ans, v)
                f[j][0] = v + j
                f[j][1] = v - j
            }
        } else {
            preMax := -0x3f3f3f3f
            for j, v := range row {
                preMax = max(preMax, f[j][0])
                res := max(v - j + preMax, v + j + sufMax[j])
                ans = max(ans, res)
                f[j][0] = res + j
                f[j][1] = res - j
            }
        }
        sufMax[n - 1] = f[n - 1][1]
        for j := n - 2; j >= 0; j -- {
            sufMax[j] = max(sufMax[j + 1], f[j][1])
        }
    }
    return int64(ans)
}
```

