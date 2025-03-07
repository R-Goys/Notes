[70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

> 假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。
>
> 每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

----

这也是个经典题目了，以前还只会递归做，哈哈。

```go
func climbStairs(n int) int {
    pre, ne := 1, 1
    for i := 2; i <= n; i ++ {
        tmp := pre + ne
        pre = ne
        ne = tmp
    }
    return ne
}
```

