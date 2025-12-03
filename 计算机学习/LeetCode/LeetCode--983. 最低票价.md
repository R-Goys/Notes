[983. 最低票价](https://leetcode.cn/problems/minimum-cost-for-tickets/)

> 在一个火车旅行很受欢迎的国度，你提前一年计划了一些火车旅行。在接下来的一年里，你要旅行的日子将以一个名为 `days` 的数组给出。每一项是一个从 `1` 到 `365` 的整数。
>
> 火车票有 **三种不同的销售方式** ：
>
> - 一张 **为期一天** 的通行证售价为 `costs[0]` 美元；
> - 一张 **为期七天** 的通行证售价为 `costs[1]` 美元；
> - 一张 **为期三十天** 的通行证售价为 `costs[2]` 美元。
>
> 通行证允许数天无限制的旅行。 例如，如果我们在第 `2` 天获得一张 **为期 7 天** 的通行证，那么我们可以连着旅行 7 天：第 `2` 天、第 `3` 天、第 `4` 天、第 `5` 天、第 `6` 天、第 `7` 天和第 `8` 天。
>
> 返回 *你想要完成在给定的列表 `days` 中列出的每一天的旅行所需要的最低消费* 。

---

这里如果根据 days 数组的长度来 dp 是不好计算的，根据 days 数组必定单增，我们需要找到最后一位元素，即需要去旅行的最后一天，根据它的长度去进行 dp ，一年最多 365 天，所以不会空间太大。

```go
func mincostTickets(days []int, costs []int) int {
    n := len(days)
    last := days[n - 1]
    isTravel := make([]bool, last + 1)
    for i := 0; i < n; i ++ {
        isTravel[days[i]] = true
    }
    f := make([]int, last + 1)
    for i := 1; i <= last; i ++ {
        if !isTravel[i] {
            f[i] = f[i - 1]
        } else {
            f[i] = min(f[i - 1] + costs[0], f[max(i - 7, 0)] + costs[1], f[max(i - 30, 0)] + costs[2])
        }
    }
    return f[last]
}
```

