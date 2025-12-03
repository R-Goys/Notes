[1423. 可获得的最大点数](https://leetcode.cn/problems/maximum-points-you-can-obtain-from-cards/)

> 几张卡牌 **排成一行**，每张卡牌都有一个对应的点数。点数由整数数组 `cardPoints` 给出。
>
> 每次行动，你可以从行的开头或者末尾拿一张卡牌，最终你必须正好拿 `k` 张卡牌。
>
> 你的点数就是你拿到手中的所有卡牌的点数之和。
>
> 给你一个整数数组 `cardPoints` 和整数 `k`，请你返回可以获得的最大点数。

---

这种找两边的最大值，反转一下就是求中间的最小值，也就是求长度为 `len(cardPoints) - k` 的最小和的子数组。

```go
func maxScore(cardPoints []int, k int) int {
    minVal := 0x3f3f3f3f
    sum := 0
    all := 0
    p := len(cardPoints) - k
    for _, x := range cardPoints {
        all += x
    }
    if p == 0 {
        return all
    }
    for i, x := range cardPoints {
        sum += x
        if i < p - 1 {
            continue
        }
        minVal = min(minVal, sum)
        sum -= cardPoints[i - p + 1]
    }
    return all - minVal
}
```

