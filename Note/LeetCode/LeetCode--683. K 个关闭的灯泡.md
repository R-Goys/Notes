[683. K 个关闭的灯泡](https://leetcode.cn/problems/k-empty-slots/)

> `n` 个灯泡排成一行，编号从 `1` 到 `n` 。最初，所有灯泡都关闭。每天 **只打开一个** 灯泡，直到 `n` 天后所有灯泡都打开。
>
> 给你一个长度为 `n` 的灯泡数组 `blubs` ，其中 `bulbs[i] = x` 意味着在第 `(i+1)` 天，我们会把在位置 `x` 的灯泡打开，其中 `i` **从 0 开始**，`x` **从 1 开始**。
>
> 给你一个整数 `k` ，请返回*恰好有两个打开的灯泡，且它们中间 **正好** 有 `k` 个 **全部关闭的** 灯泡的 **最小的天数*** 。*如果不存在这种情况，返回 `-1` 。*

---

奇怪的题目，感觉没什么定长滑动窗口的部分，对于每次遍历到的灯泡，我们不仅要检查后面是否存在，还需要检查前面是否存在这样的灯泡。

```go
func kEmptySlots(bulbs []int, k int) int {
    n := len(bulbs)
    src := make([]bool, n + 1)
    var check func(i int) bool
    check = func(x int) bool {
        if src[x] == false || src[x - k - 1] == false {
            return false
        }
        for m := x - k; m < x; m ++ {
            if src[m] == true {
                return false
            }
        }
        return true
    }
    for i, x := range bulbs {
        src[x] = true

        if (x >= k + 1 && check(x)) || (x + k + 1 <= n && check(x + k + 1)) {
            return i + 1
        }
    }
    return -1
}
```

