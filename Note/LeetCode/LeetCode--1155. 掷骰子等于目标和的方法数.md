[1155. 掷骰子等于目标和的方法数](https://leetcode.cn/problems/number-of-dice-rolls-with-target-sum/)

> 这里有 `n` 个一样的骰子，每个骰子上都有 `k` 个面，分别标号为 `1` 到 `k` 。
>
> 给定三个整数 `n`、`k` 和 `target`，请返回投掷骰子的所有可能得到的结果（共有 `kn` 种方式），使得骰子面朝上的数字总和等于 `target`。
>
> 由于答案可能很大，你需要对 `109 + 7` **取模**。

---

分组背包，感觉没办法优化成一维的：因为会有重复的计算。

```go
func numRollsToTarget(n int, k int, target int) int {
    f := make([][]int, n + 1)
    for i := range f {
        f[i] = make([]int, target + 1)
    }
    mod := 1000000000 + 7
    f[0][0] = 1
    for i := 1; i <= n; i ++ {
        for j := 0; j <= target; j ++ {
            for m := 1; m <= j && m <= k; m ++ {
                f[i][j] = (f[i][j] + f[i - 1][j - m]) % mod
            }
        }
    }
    return f[n][target]
}
```

仔细思考了一下，我们可以在计算当前行之前将数据清零，以此来实现空间优化：

```go
func numRollsToTarget(n int, k int, target int) int {
    mod := 1000000000 + 7

    dpPrev := make([]int, target+1)
    dpCurr := make([]int, target+1)
    dpPrev[0] = 1
    
    for i := 1; i <= n; i++ {
        // 清零
        for j := 0; j <= target; j++ {
            dpCurr[j] = 0
        }
        for j := 1; j <= target; j++ {
            for m := 1; m <= k && m <= j; m++ {
                dpCurr[j] = (dpCurr[j] + dpPrev[j-m]) % mod
            }
        }
        dpPrev, dpCurr = dpCurr, dpPrev
    }
    return dpPrev[target]
}
```

