[3183. 达到总和的方法数量](https://leetcode.cn/problems/the-number-of-ways-to-make-the-sum/)

> 给定 **无限** 数量的面值为 1，2，6 的硬币，并且 **只有** 2 枚硬币面值为 4。
>
> 给定一个整数 `n` ，返回用你持有的硬币达到总和 `n` 的方法数量。
>
> 因为答案可能会很大，将其 **取模** `109 + 7`。
>
> **注意** 硬币的顺序并不重要，`[2, 2, 3]` 与 `[2, 3, 2]` 相同。

---

easy，两个多余的硬币特殊处理一下就行了

```go
var src = []int{1, 2, 6}

func numberOfWays(n int) int {
    mod := 1000000000 + 7
    f := make([]int, n + 1)
    if n >= 4 {
        f[4] = 1
    }
    if n >= 8 {
        f[8] = 1
    }
    f[0] = 1
    for i := 0; i < 3; i ++ {
        for j := src[i]; j <= n; j ++ {
            f[j] = (f[j] + f[j - src[i]]) % mod
        }
    }
    return f[n]
}
```

