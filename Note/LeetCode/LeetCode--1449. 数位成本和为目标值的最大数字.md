[1449. 数位成本和为目标值的最大数字](https://leetcode.cn/problems/form-largest-integer-with-digits-that-add-up-to-target/)

> 给你一个整数数组 `cost` 和一个整数 `target` 。请你返回满足如下规则可以得到的 **最大** 整数：
>
> - 给当前结果添加一个数位（`i + 1`）的成本为 `cost[i]` （`cost` 数组下标从 0 开始）。
> - 总成本必须恰好等于 `target` 。
> - 添加的数位中没有数字 0 。
>
> 由于答案可能会很大，请你以字符串形式返回。
>
> 如果按照上述要求无法得到任何整数，请你返回 "0" 。

---

先利用完全背包思路计算出最长的长度，然后倒序遍历 0~9 ，找到找到符合长度并且符合成本的数字，此时一定是最大的数字。
```go
func largestNumber(cost []int, target int) string {
    n := len(cost)
    f := make([]int, target + 1)
    for i := 0; i <= target; i ++ {
        f[i] = -0x3f3f3f3f
    }
    f[0] = 0
    for i := 0; i < n; i ++ {
        for j := cost[i]; j <= target; j ++ {
            f[j] = max(f[j], f[j - cost[i]] + 1)
        }
    }
    if f[target] < 0 {
        return "0"
    }
    ans := []byte{}
    for i, j := n - 1, target; i >= 0; i -- {
        for j >= cost[i] && f[j] == f[j - cost[i]] + 1 {
            ans = append(ans, byte(i + '1'))
            j -= cost[i]
        }
    }
    return string(ans)
}
```

