[2787. 将一个数字表示成幂的和的方案数](https://leetcode.cn/problems/ways-to-express-an-integer-as-sum-of-powers/)

> 给你两个 **正** 整数 `n` 和 `x` 。
>
> 请你返回将 `n` 表示成一些 **互不相同** 正整数的 `x` 次幂之和的方案数。换句话说，你需要返回互不相同整数 `[n1, n2, ..., nk]` 的集合数目，满足 `n = n1x + n2x + ... + nkx` 。
>
> 由于答案可能非常大，请你将它对 `109 + 7` 取余后返回。
>
> 比方说，`n = 160` 且 `x = 3` ，一个表示 `n` 的方法是 `n = 23 + 33 + 53` 。

---

先计算背包的元素，然后就是背包问题：

```go
func numberOfWays(n int, x int) int {
    src := make([]int, n + 1)
    Len := n
    mod := 1000000007
    f := make([]int, n + 1)
    for i := 1; i <= n; i ++ {
        src[i] = 1
        for j := 0; j < x; j ++ {
            src[i] *= i
        }
        if f[i] > n {
            src = src[:i - 1]
            Len = i - 1
            break
        }
    }
    f[0] = 1
    for i := 1; i <= Len; i ++ {
        for j := n; j >= src[i]; j -- {
            f[j] = (f[j] + f[j - src[i]]) % mod
        }
    }
    return f[n] % mod
}
```

这里也可以用快速幂计算，但是数据量很小，所以暴力也不会超时