[3296. 移山所需的最少秒数](https://leetcode.cn/problems/minimum-number-of-seconds-to-make-mountain-height-zero/)

> 给你一个整数 `mountainHeight` 表示山的高度。
>
> 同时给你一个整数数组 `workerTimes`，表示工人们的工作时间（单位：**秒**）。
>
> 工人们需要 **同时** 进行工作以 **降低** 山的高度。对于工人 `i` :
>
> - 山的高度降低 `x`，需要花费 `workerTimes[i] + workerTimes[i] * 2 + ... + workerTimes[i] * x` 秒。例如：
>   - 山的高度降低 1，需要 `workerTimes[i]` 秒。
>   - 山的高度降低 2，需要 `workerTimes[i] + workerTimes[i] * 2` 秒，依此类推。
>
> 返回一个整数，表示工人们使山的高度降低到 0 所需的 **最少** 秒数。

---

跟前几题一样，唯一需要注意的就是计算麻烦，跟数学题一样。

```go
func minNumberOfSeconds(mountainHeight int, workerTimes []int) int64 {
    h := (mountainHeight -1 ) / len(workerTimes) + 1
    maxn := -1
    for _, x := range workerTimes {
        maxn = max(x, maxn)
    }
    top := maxn * h * (h + 1) / 2
    time := sort.Search(top - 1, func(i int) bool {
        return move(workerTimes, i + 1) >= mountainHeight
    })  
    return int64(time + 1)
}

func move(workerTimes []int, time int) int {
    Height := 0
    for _, x := range workerTimes {
        Height += (int(math.Sqrt(float64(time / x * 8 + 1))) - 1) / 2
    }
    return Height
}
```

