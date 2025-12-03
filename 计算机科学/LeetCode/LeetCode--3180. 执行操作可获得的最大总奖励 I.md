[3180. 执行操作可获得的最大总奖励 I](https://leetcode.cn/problems/maximum-total-reward-using-operations-i/)

> 给你一个整数数组 `rewardValues`，长度为 `n`，代表奖励的值。
>
> 最初，你的总奖励 `x` 为 0，所有下标都是 **未标记** 的。你可以执行以下操作 **任意次** ：
>
> - 从区间 `[0, n - 1]` 中选择一个 **未标记** 的下标 `i`。
> - 如果 `rewardValues[i]` **大于** 你当前的总奖励 `x`，则将 `rewardValues[i]` 加到 `x` 上（即 `x = x + rewardValues[i]`），并 **标记** 下标 `i`。
>
> 以整数形式返回执行最优操作能够获得的 **最大** 总奖励。

---

感觉这道题有点奇怪，由于每一次只能加上比当前总和大的数字，所以我们的最后最大的总奖励一定是小于 `2 * max(src...)` 的，我们可以通过标记来确定是否存在这个数量的奖励总和，然后我们最终可以从后往前遍历，找到第一个标记为奖励的就是最大值。

同时我们还需要原数组是一个递增的数组，以便于我们从小到大遍历。

```go
func maxTotalReward(rewardValues []int) int {
    sort.Ints(rewardValues)
    n := rewardValues[len(rewardValues) - 1]
    f := make([]int, 2 * n)
    f[0] = 1
    for _, x := range rewardValues {
        for k := 2 * x - 1; k >= x; k -- {
            if f[k - x] == 1 {
                f[k] = 1
            }
        }
    }
    for i := len(f) - 1; i >= 0; i -- {
        if f[i] == 1 {
            return i
        }
    }
    return 0
}
```

