[746. 使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)

> 给你一个整数数组 `cost` ，其中 `cost[i]` 是从楼梯第 `i` 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。
>
> 你可以选择从下标为 `0` 或下标为 `1` 的台阶开始爬楼梯。
>
> 请你计算并返回达到楼梯顶部的最低花费。

---

秒，无意义的题目。

```go
func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    p1, p2 := cost[0], cost[1]
    for i := 2; i < n; i ++ {
        p2, p1 = min(p1, p2) + cost[i], p2
    }
    return min(p1, p2)
}
```

