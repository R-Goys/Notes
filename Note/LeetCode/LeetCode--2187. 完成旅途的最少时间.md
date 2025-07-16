[2187. 完成旅途的最少时间](https://leetcode.cn/problems/minimum-time-to-complete-trips/)

> 给你一个数组 `time` ，其中 `time[i]` 表示第 `i` 辆公交车完成 **一趟****旅途** 所需要花费的时间。
>
> 每辆公交车可以 **连续** 完成多趟旅途，也就是说，一辆公交车当前旅途完成后，可以 **立马开始** 下一趟旅途。每辆公交车 **独立** 运行，也就是说可以同时有多辆公交车在运行且互不影响。
>
> 给你一个整数 `totalTrips` ，表示所有公交车 **总共** 需要完成的旅途数目。请你返回完成 **至少** `totalTrips` 趟旅途需要花费的 **最少** 时间。

---

求和 + 二分，和上一题一样

```go
func minimumTime(time []int, totalTrips int) int64 {
    maxn := 0
    for _, x := range time {
        maxn = max(maxn, x)
    }
    maxn *= totalTrips
    div := sort.Search(maxn, func(i int) bool {
        return sum(time, i) >= totalTrips
    })
    return int64(div)
}

func sum(nums []int, div int) int {
    ans := 0
    for _, x := range nums {
        ans += div / x
    }
    return ans
}
```

