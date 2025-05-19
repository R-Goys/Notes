[3290. 最高乘法得分](https://leetcode.cn/problems/maximum-multiplication-score/)

> 给你一个大小为 4 的整数数组 `a` 和一个大小 **至少**为 4 的整数数组 `b`。
>
> 你需要从数组 `b` 中选择四个下标 `i0`, `i1`, `i2`, 和 `i3`，并满足 `i0 < i1 < i2 < i3`。你的得分将是 `a[0] * b[i0] + a[1] * b[i1] + a[2] * b[i2] + a[3] * b[i3]` 的值。
>
> 返回你能够获得的 **最大** 得分。

---

每一个状态根据上一层地状态来转移，max 里面的参数对应着选当前的数字和不选当前的数字：

```go
func maxScore(a []int, b []int) int64 {
    n := len(b)
    f := make([][5]int, n + 1)
	for j := 1; j < 5; j ++ {
		f[0][j] = math.MinInt64 / 2
	}
    for i := 1; i <= n; i ++ {
        for j := 1; j <= 4; j ++ {
            f[i][j] = max(f[i - 1][j - 1] + a[j - 1] * b[i - 1], f[i - 1][j])
        }
    }
    return int64(f[n][4])
}
```

在这基础上可以进行空间优化，当然，这里需要进行反序遍历来保证上一个状态在计算之前不会被覆盖。

```go
func maxScore(a []int, b []int) int64 {
    n := len(b)
    f := [5]int{}
	for j := 1; j < 5; j ++ {
		f[j] = math.MinInt64 / 2
	}
    for i := 1; i <= n; i ++ {
        for j := 4; j >= 1; j -- {
            f[j] = max(f[j], f[j - 1] + a[j - 1] * b[i - 1])
        }
    }
    return int64(f[4])
}
```

