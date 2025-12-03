[509. 斐波那契数](https://leetcode.cn/problems/fibonacci-number/)

> **斐波那契数** （通常用 `F(n)` 表示）形成的序列称为 **斐波那契数列** 。该数列由 `0` 和 `1` 开始，后面的每一项数字都是前面两项数字的和。也就是：
>
> ```
> F(0) = 0，F(1) = 1
> F(n) = F(n - 1) + F(n - 2)，其中 n > 1
> ```
>
> 给定 `n` ，请计算 `F(n)` 。

---

秒杀

```go
func fib(n int) int {
    a1 := 0
    a2 := 1
    for i := 0; i < n; i ++ {
        a2, a1 = a1 + a2, a2
    }
    return a1
}
```

