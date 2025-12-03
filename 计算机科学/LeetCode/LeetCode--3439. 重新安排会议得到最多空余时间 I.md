[3439. 重新安排会议得到最多空余时间 I](https://leetcode.cn/problems/reschedule-meetings-for-maximum-free-time-i/)

> 给你一个整数 `eventTime` 表示一个活动的总时长，这个活动开始于 `t = 0` ，结束于 `t = eventTime` 。
>
> 同时给你两个长度为 `n` 的整数数组 `startTime` 和 `endTime` 。它们表示这次活动中 `n` 个时间 **没有重叠** 的会议，其中第 `i` 个会议的时间为 `[startTime[i], endTime[i]]` 。
>
> 你可以重新安排 **至多** `k` 个会议，安排的规则是将会议时间平移，且保持原来的 **会议时长** ，你的目的是移动会议后 **最大化** 相邻两个会议之间的 **最长** 连续空余时间。
>
> 移动前后所有会议之间的 **相对** 顺序需要保持不变，而且会议时间也需要保持互不重叠。
>
> 请你返回重新安排会议以后，可以得到的 **最大** 空余时间。
>
> **注意**，会议 **不能** 安排到整个活动的时间以外。

---

翻译过来就是求 k 个会议中有多少最大空闲时间，我们可以重构 startTime 和 endTime，增加两个哨兵元素来判断，然后滑动窗口计算我们在 k 个会议中可以有多少空闲时间，为什么需要两个哨兵元素？因为比如我们 k = 1 的时候，我们的空闲的时间就是 0 ~ startTime 和 endTime ~ NextStartTime，所以为了正确计算这两个事件，就需要哨兵元素。

```go
func maxFreeTime(eventTime int, k int, startTime []int, endTime []int) int {
    startTime = append([]int{0}, startTime...)
    startTime = append(startTime, eventTime)
    endTime = append([]int{0}, endTime...)
    endTime = append(endTime, eventTime)
    n := len(startTime)
    rests := make([]int, 0)
    rest := 0
    ans := 0
    for i := 1; i < n; i ++ {
        rests = append(rests, startTime[i] - endTime[i - 1])
    }
    for i := 0; i < n - 1; i ++ {
        rest += rests[i]
        if i < k {
            continue
        }
        ans = max(rest, ans)
        rest -= rests[i - k]
    }
    return ans
}
```

