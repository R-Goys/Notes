[343. 整数拆分](https://leetcode.cn/problems/integer-break/)

> 给定一个正整数 `n` ，将其拆分为 `k` 个 **正整数** 的和（ `k >= 2` ），并使这些整数的乘积最大化。
>
> 返回 *你可以获得的最大乘积* 。

---

整数划分，难点在于想出状态转移方程，代码是很简单的：

```go
func integerBreak(n int) int {
    f := make([]int, n + 1)
    for i := 2; i <= n; i ++ {
        curMax := 0
        for j := 1; j < i; j ++ {
            curMax = max(curMax, max(j * (i - j), j * f[i - j]))
        }
        f[i] = curMax
    }
    return f[n]
}
```

关键点就是在于嵌套的max的计算的是什么，先将当前的max计算出来，由于k可以是任意个数，所以我们只需要最多分割到i个数字即可。