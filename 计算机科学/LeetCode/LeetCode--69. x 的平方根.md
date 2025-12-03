[69. x 的平方根 ](https://leetcode.cn/problems/sqrtx/)

> 给你一个非负整数 `x` ，计算并返回 `x` 的 **算术平方根** 。
>
> 由于返回类型是整数，结果只保留 **整数部分** ，小数部分将被 **舍去 。**
>
> **注意：**不允许使用任何内置指数函数和算符，例如 `pow(x, 0.5)` 或者 `x ** 0.5` 。

----

二分查找

```go
func mySqrt(x int) int {
    ans := -1
	l, r := 0, x
    for l <= r {
        mid := (l + r) / 2
        if mid * mid > x {
            r = mid - 1
        } else {
            ans = mid
            l = mid + 1
        }
    }
    return ans
}
```

