[3393. 统计异或值为给定值的路径数目](https://leetcode.cn/problems/count-paths-with-the-given-xor-value/)

> 给你一个大小为 `m x n` 的二维整数数组 `grid` 和一个整数 `k` 。
>
> 你的任务是统计满足以下 **条件** 且从左上格子 `(0, 0)` 出发到达右下格子 `(m - 1, n - 1)` 的路径数目：
>
> - 每一步你可以向右或者向下走，也就是如果格子存在的话，可以从格子 `(i, j)` 走到格子 `(i, j + 1)` 或者格子 `(i + 1, j)` 。
> - 路径上经过的所有数字 `XOR` 异或值必须 **等于** `k` 。
>
> 请你返回满足上述条件的路径总数。
>
> 由于答案可能很大，请你将答案对 `10^9 + 7` **取余** 后返回。

---

网格 dp ，很明显二维的数组不能满足我们的要求，很多人第一眼都是直接暴搜，但是这道题暴搜会超时，我们只能采取 dp 的做法，首先开辟三维数组，第三个索引表示我们当前的异或值，由于我们的异或值题目中给出了一定限制，我们可以直接开辟 16 个空间，每次的**当前异或值**可以直接枚举前缀然后与当前元素异或运算来得到：

```go
func countPathsWithXorValue(grid [][]int, k int) int {
    n, m := len(grid), len(grid[0])
    mod := 1000000007
    f := make([][][16]int, n)
    for i := 0 ; i < n ;i ++ {
        f[i] = make([][16]int, m)
    }
    f[0][0][grid[0][0]] = 1
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            for t := 0; t < 16; t ++ {
                curXor := t ^ grid[i][j]

                if i > 0 {
                    f[i][j][curXor] = f[i - 1][j][t]
                }

                if j > 0 {
                    f[i][j][curXor] += f[i][j - 1][t]
                    f[i][j][curXor] %= mod
                }
            }
        }
    }
    return f[n - 1][m - 1][k]
}
```

