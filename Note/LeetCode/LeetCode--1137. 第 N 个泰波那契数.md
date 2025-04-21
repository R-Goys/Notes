[1137. 第 N 个泰波那契数](https://leetcode.cn/problems/n-th-tribonacci-number/)

> 泰波那契序列 Tn 定义如下： 
>
> T0 = 0, T1 = 1, T2 = 1, 且在 n >= 0 的条件下 Tn+3 = Tn + Tn+1 + Tn+2
>
> 给你整数 `n`，请返回第 n 个泰波那契数 Tn 的值。

很简单的一题，用来复活再适合不过

```go
func tribonacci(n int) int {
    j, k, m := 0, 1, 1
    for i := 0; i < n; i ++ {
        j, k, m = k, m, k + m + j
    }
    return j    
}
```

