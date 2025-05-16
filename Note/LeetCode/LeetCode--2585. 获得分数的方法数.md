[2585. 获得分数的方法数](https://leetcode.cn/problems/number-of-ways-to-earn-points/)

> 考试中有 `n` 种类型的题目。给你一个整数 `target` 和一个下标从 **0** 开始的二维整数数组 `types` ，其中 `types[i] = [counti, marksi] `表示第 `i` 种类型的题目有 `counti` 道，每道题目对应 `marksi` 分。
>
> 返回你在考试中恰好得到 `target` 分的方法数。由于答案可能很大，结果需要对 `109 +7` 取余。
>
> **注意**，同类型题目无法区分。
>
> - 比如说，如果有 `3` 道同类型题目，那么解答第 `1` 和第 `2` 道题目与解答第 `1` 和第 `3` 道题目或者第 `2` 和第 `3` 道题目是相同的。

---

多重背包问题，相比于 01 背包，多了一个枚举每个种类的元素的数量的步骤：

```go
func waysToReachTarget(target int, types [][]int) int {
    n := len(types)
    mod := 1000000000 + 7
    f := make([]int, target + 1)
    f[0] = 1 
    for i := 0; i < n; i ++ {
        for j := target; j > 0; j -- {
            for k := 1; k <= min(types[i][0], j / types[i][1]); k ++ {
                f[j] = (f[j] + f[j - k * types[i][1]]) % mod
            }
        }
    }
    return f[target]
}
```

