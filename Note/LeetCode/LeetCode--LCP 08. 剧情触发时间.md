[LCP 08. 剧情触发时间](https://leetcode.cn/problems/ju-qing-hong-fa-shi-jian/)

> 在战略游戏中，玩家往往需要发展自己的势力来触发各种新的剧情。一个势力的主要属性有三种，分别是文明等级（`C`），资源储备（`R`）以及人口数量（`H`）。在游戏开始时（第 0 天），三种属性的值均为 0。
>
> 随着游戏进程的进行，每一天玩家的三种属性都会对应**增加**，我们用一个二维数组 `increase` 来表示每天的增加情况。这个二维数组的每个元素是一个长度为 3 的一维数组，例如 `[[1,2,1],[3,4,2]]` 表示第一天三种属性分别增加 `1,2,1` 而第二天分别增加 `3,4,2`。
>
> 所有剧情的触发条件也用一个二维数组 `requirements` 表示。这个二维数组的每个元素是一个长度为 3 的一维数组，对于某个剧情的触发条件 `c[i], r[i], h[i]`，如果当前 `C >= c[i]` 且 `R >= r[i]` 且 `H >= h[i]` ，则剧情会被触发。
>
> 根据所给信息，请计算每个剧情的触发时间，并以一个数组返回。如果某个剧情不会被触发，则该剧情对应的触发时间为 -1 。

---

前缀和 + 二分查找

```go
func getTriggerTime(increase [][]int, requirements [][]int) []int {
    prefix := make([][3]int, len(increase) + 1)
    for i, unit := range increase {
        prefix[i + 1][0] = unit[0] + prefix[i][0]
        prefix[i + 1][1] = unit[1] + prefix[i][1]
        prefix[i + 1][2] = unit[2] + prefix[i][2]
    }
    ans := make([]int, 0)
    for _, x := range requirements {
        idx := sort.Search(len(prefix), func(i int) bool {
            return prefix[i][0] >= x[0] && prefix[i][1] >= x[1] && prefix[i][2] >= x[2]
        })
        if idx == len(prefix) {
            idx = -1
        }
        ans = append(ans, idx)
    }
    return ans
}
```

