[1124. 表现良好的最长时间段](https://leetcode.cn/problems/longest-well-performing-interval/)

> 给你一份工作时间表 `hours`，上面记录着某一位员工每天的工作小时数。
>
> 我们认为当员工一天中的工作小时数大于 `8` 小时的时候，那么这一天就是「**劳累的一天**」。
>
> 所谓「表现良好的时间段」，意味在这段时间内，「劳累的天数」是严格 **大于**「不劳累的天数」。
>
> 请你返回「表现良好时间段」的最大长度。

---

牛逼，感觉一辈子都想不出这种前缀和+栈的解法

```go
func longestWPI(hours []int) int {
    cnt := 0
    pos := make([]int, len(hours)+2)
    ans := 0
    for i, x := range hours {
        i ++
        if x > 8 {
            cnt --
        } else {
            cnt ++
        }
        if cnt < 0 {
            ans = i
        } else {
            if pos[cnt + 1] > 0 {
                ans = max(ans, i - pos[cnt + 1])
            }
            if pos[cnt] == 0 {
                pos[cnt] = i
            }
        }
    }
    return ans
}
```

