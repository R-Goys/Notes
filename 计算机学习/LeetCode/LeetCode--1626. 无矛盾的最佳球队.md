[1626. 无矛盾的最佳球队](https://leetcode.cn/problems/best-team-with-no-conflicts/)

> 假设你是球队的经理。对于即将到来的锦标赛，你想组合一支总体得分最高的球队。球队的得分是球队中所有球员的分数 **总和** 。
>
> 然而，球队中的矛盾会限制球员的发挥，所以必须选出一支 **没有矛盾** 的球队。如果一名年龄较小球员的分数 **严格大于** 一名年龄较大的球员，则存在矛盾。同龄球员之间不会发生矛盾。
>
> 给你两个列表 `scores` 和 `ages`，其中每组 `scores[i]` 和 `ages[i]` 表示第 `i` 名球员的分数和年龄。请你返回 **所有可能的无矛盾球队中得分最高那支的分数** 。

---

对于这道题，我们简单的想一想，我们能否直接通过线性 dp 结合结合类似背包问题来做？结论是不行，首先，我们如果直接判断没有矛盾的条件来进行 dp，会导致问题，比如当我们前面一个状态是加入了一个年龄比较大的球员，如果我们之后有加入年龄较小，但是是最优解，此时却因为年龄问题而无法正确的 dp。

所以正确的做法是先按照年龄排序，然后再进行 dp。

```go
func bestTeamScore(scores []int, ages []int) int {
    n := len(scores)
    f := make([]int, n)
    ans := 0
    src := make([][2]int, n)

    for i := 0; i < n; i++ {
        src[i] = [2]int{ages[i], scores[i]}
    }

    sort.Slice(src, func(i, j int) bool {
        if src[i][0] == src[j][0] {
            return src[i][1] < src[j][1]
        }
        return src[i][0] < src[j][0]
    })

    for i := 0; i < n; i ++ {
        f[i] = src[i][1]
        for j := 0; j < i; j ++ {
            if src[j][1] <= src[i][1] {
                f[i] = max(f[i], f[j] + src[i][1])
            }
        } 
        ans = max(f[i], ans)
    }
    return ans
}
```

