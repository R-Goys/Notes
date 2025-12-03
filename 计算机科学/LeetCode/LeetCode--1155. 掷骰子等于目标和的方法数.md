[1155. 掷骰子等于目标和的方法数](https://leetcode.cn/problems/number-of-dice-rolls-with-target-sum/)

> 这里有 `n` 个一样的骰子，每个骰子上都有 `k` 个面，分别标号为 `1` 到 `k` 。
>
> 给定三个整数 `n`、`k` 和 `target`，请返回投掷骰子的所有可能得到的结果（共有 `kn` 种方式），使得骰子面朝上的数字总和等于 `target`。
>
> 由于答案可能很大，你需要对 `109 + 7` **取模**。

---

分组背包，通过递推公式 $$f(i, m) = f(i - 1, m - 1) + ...+f(i - 1, m - k)$$ 可以推出下面的代码，感觉没办法优化成一维的，因为会有重复的计算。

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

灵神终究是神，贴一下灵神的题解，思路是先直接计算前缀和，这里灵神讲的很清晰，但是我也用自己的话说一下。

我们可以通过状态转移方程来知道 f[i\][m] 的公式，然后我们会发现规律 $$f(i, m) = f(i - 1, m - 1) + ...+f(i - 1, m - k)$$ 我们可以将 f[i\][m] 这个式子转换为前缀和的形式: $$s[j]=s[j−1]+f[i−1][j]$$  

然后我们根据这个前缀和公式，可以轻易地计算每一个状态，当我们地目标和小于 k 的时候，自然就沿用这个前缀和，当目标和大于 k 的时候，就是用前缀和计算即可：

这里的 f[i - 1] 最开始就是我们的上一个状态，而第一步循环完成之后，我们就得到了前面状态的前缀和，通过它，我们可以直接计算出当前的状态值。

```go
func numRollsToTarget(n int, k int, target int) int {
    mod := 1000000000 + 7

    f := make([]int, target - n + 1)
    f[0] = 1
    for i := 1; i <= n; i ++ {
        maxJ := min(i * (k - 1), target - n)
        for j := 1; j <= maxJ; j ++ {
            f[j] += f[j - 1]
        }
        for j := maxJ; j >= k; j -- {
            f[j] = (f[j] - f[j - k]) % mod
        }
    }
    return f[target-n]
}
```

