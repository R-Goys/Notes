[1964. 找出到每个位置为止最长的有效障碍赛跑路线](https://leetcode.cn/problems/find-the-longest-valid-obstacle-course-at-each-position/)

> 你打算构建一些障碍赛跑路线。给你一个 **下标从 0 开始** 的整数数组 `obstacles` ，数组长度为 `n` ，其中 `obstacles[i]` 表示第 `i` 个障碍的高度。
>
> 对于每个介于 `0` 和 `n - 1` 之间（包含 `0` 和 `n - 1`）的下标 `i` ，在满足下述条件的前提下，请你找出 `obstacles` 能构成的最长障碍路线的长度：
>
> - 你可以选择下标介于 `0` 到 `i` 之间（包含 `0` 和 `i`）的任意个障碍。
> - 在这条路线中，必须包含第 `i` 个障碍。
> - 你必须按障碍在 `obstacles` 中的 **出现顺序** 布置这些障碍。
> - 除第一个障碍外，路线中每个障碍的高度都必须和前一个障碍 **相同** 或者 **更高** 。
>
> 返回长度为 `n` 的答案数组 `ans` ，其中 `ans[i]` 是上面所述的下标 `i` 对应的最长障碍赛跑路线的长度。

---

根据题意，我们需要找到每一个前缀里面的最长非降序的子序列，并且这个序列需要包含这个前缀的最后一个数字，很明显，我们可以遍历每个元素，进行二分，但是如何保证最后一个数字能够包含在子序列里面呢？很简单，我们每次二分记录的插入位置，就是我们的长度了。

由于我们此时需要的子序列不是之前的严格上升子序列，所以search的条件会发生改变。

```go
func longestObstacleCourseAtEachPosition(obstacles []int) []int {
    ans := make([]int, 0)
    f := make([]int, 0)
    for i := 0; i < len(obstacles); i ++ {
        dst := sort.SearchInts(f, obstacles[i] + 1)
        if dst < len(f) {
            f[dst] = obstacles[i]
        } else {
            f = append(f, obstacles[i])
        }
        ans = append(ans, dst + 1)
    }
    return ans
}
```

