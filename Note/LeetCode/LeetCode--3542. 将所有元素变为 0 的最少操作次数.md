[3542. 将所有元素变为 0 的最少操作次数](https://leetcode.cn/problems/minimum-operations-to-convert-all-elements-to-zero/)

> 给你一个大小为 `n` 的 **非负** 整数数组 `nums` 。你的任务是对该数组执行若干次（可能为 0 次）操作，使得 **所有** 元素都变为 0。
>
> 在一次操作中，你可以选择一个子数组 `[i, j]`（其中 `0 <= i <= j < n`），将该子数组中所有 **最小的非负整数** 的设为 0。
>
> 返回使整个数组变为 0 所需的**最少**操作次数。
>
> 一个 **子数组** 是数组中的一段连续元素。

---

依旧单调栈，可以比作是一个图形，峰尖必定会操作一次，而最下面的会合并到一起。针对于一个山峰，我们可以自底向上的进行操作，保证只会进行必要的操作。

```go
func minOperations(nums []int) int {
    st := make([]int, 0)
    ans := 0
    for _, x := range nums {
        for len(st) > 0 && x < st[len(st) - 1] {
            st = st[:len(st) - 1]
            ans ++
        }
        if len(st) == 0 || x != st[len(st) - 1] {
            st = append(st, x)
        }
    }
    if st[0] == 0 {
        ans --
    }
    return ans + len(st)
}
```

