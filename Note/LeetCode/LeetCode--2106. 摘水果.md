[2106. 摘水果](https://leetcode.cn/problems/maximum-fruits-harvested-after-at-most-k-steps/)

> 在一个无限的 x 坐标轴上，有许多水果分布在其中某些位置。给你一个二维整数数组 `fruits` ，其中 `fruits[i] = [positioni, amounti]` 表示共有 `amounti` 个水果放置在 `positioni` 上。`fruits` 已经按 `positioni` **升序排列** ，每个 `positioni` **互不相同** 。
>
> 另给你两个整数 `startPos` 和 `k` 。最初，你位于 `startPos` 。从任何位置，你可以选择 **向左或者向右** 走。在 x 轴上每移动 **一个单位** ，就记作 **一步** 。你总共可以走 **最多** `k` 步。你每达到一个位置，都会摘掉全部的水果，水果也将从该位置消失（不会再生）。
>
> 返回你可以摘到水果的 **最大总数** 。

---

难点在于从 startPos 开始，既可以向右，也可以向左移动，导致滑窗很困难，事实上，我们可以利用 startPos 限制的区间来尽可能地让滑窗更简单，我们只需要指明我们需要的区间，然后通过 startPos 限制的区间，以及 k 来验证当前区间是否可行，然后计算即可。

```go
func maxTotalFruits(fruits [][]int, startPos int, k int) int {
    n := len(fruits)

    left := sort.Search(n, func(i int) bool {
        return fruits[i][0] >= startPos - k
    })
    right, sum := left, 0
    ans := 0
    for ; right < n && fruits[right][0] <= startPos + k; right ++ {
        sum += fruits[right][1]
        // 这里表示的是移动的距离 > k，则需要移动 left
        for fruits[right][0] * 2 - fruits[left][0] - startPos > k &&
            fruits[right][0] - fruits[left][0] * 2 + startPos > k {
            sum -= fruits[left][1]
            left ++
        }
        ans = max(ans, sum)
    }
    return ans
}
```

