[1176. 健身计划评估](https://leetcode.cn/problems/diet-plan-performance/)

> 你的好友是一位健身爱好者。前段日子，他给自己制定了一份健身计划。现在想请你帮他评估一下这份计划是否合理。
>
> 他会有一份计划消耗的卡路里表，其中 `calories[i]` 给出了你的这位好友在第 `i` 天需要消耗的卡路里总量。
>
> 为了更好地评估这份计划，对于卡路里表中的每一天，你都需要计算他 「这一天以及之后的连续几天」 （共 `k` 天）内消耗的总卡路里 *T：*
>
> - 如果 `T < lower`，那么这份计划相对糟糕，并失去 1 分； 
> - 如果 `T > upper`，那么这份计划相对优秀，并获得 1 分；
> - 否则，这份计划普普通通，分值不做变动。
>
> 请返回统计完所有 `calories.length` 天后得到的总分作为评估结果。
>
> 注意：总分可能是负数。

---

题目还有给的样例很弱智，描述一点都不清晰。。。

这道题是让我们针对于数组中每个长度为 k 的子数组求和，如果总和大于 upper，而 score + 1，小于 lower，而 score - 1，否则不变化。

```go
func dietPlanPerformance(calories []int, k int, lower int, upper int) int {
    sum := 0
    point := 0
    for i, x := range calories {
        sum += x
        if i < k - 1 {
            continue
        }
        if sum < lower {
            point --
        } else if sum > upper {
            point ++
        }
        sum -= calories[i - k + 1]
    }
    return point
}
```

