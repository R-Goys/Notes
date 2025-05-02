[2140. 解决智力问题](https://leetcode.cn/problems/solving-questions-with-brainpower/)

> 给你一个下标从 **0** 开始的二维整数数组 `questions` ，其中 `questions[i] = [pointsi, brainpoweri]` 。
>
> 这个数组表示一场考试里的一系列题目，你需要 **按顺序** （也就是从问题 `0` 开始依次解决），针对每个问题选择 **解决** 或者 **跳过** 操作。解决问题 `i` 将让你 **获得** `pointsi` 的分数，但是你将 **无法** 解决接下来的 `brainpoweri` 个问题（即只能跳过接下来的 `brainpoweri` 个问题）。如果你跳过问题 `i` ，你可以对下一个问题决定使用哪种操作。
>
> - 比方说，给你 `questions = [[3, 2], [4, 3], [4, 4], [2, 5]]` ：
>
>   - 如果问题 `0` 被解决了， 那么你可以获得 `3` 分，但你不能解决问题 `1` 和 `2` 。
>   - 如果你跳过问题 `0` ，且解决问题 `1` ，你将获得 `4` 分但是不能解决问题 `2` 和 `3` 。
> 
>请你返回这场考试里你能获得的 **最高** 分数。

---

这道题很简单，从后往前进行状态转移，先计算只考虑在第 i 个元素及之后可以得到的最高分，然后从后往前进行dp，可以轻松的根据之前的状态计算出考虑第 0 个元素及之后的最大值。

```go
func mostPoints(questions [][]int) int64 {
    n := len(questions)
    f := make([]int, n + 1)
    for i := n - 1; i >= 0; i-- {
        f[i] = max(f[i + 1], questions[i][0] + f[min(n, i + questions[i][1] + 1)])
    }
    return int64(f[0])
}
```

二刷，注意只能从跳过的元素的下一个元素开始计算，需要 + 1 。

```go
func mostPoints(questions [][]int) int64 {
    n := len(questions)
    f := make([]int, n + 1)
    for i := n - 1; i >= 0; i -- {
        f[i] = max(f[i + 1], f[min(n, i + questions[i][1] + 1)] + questions[i][0])
    }
    return int64(f[0])
}
```

