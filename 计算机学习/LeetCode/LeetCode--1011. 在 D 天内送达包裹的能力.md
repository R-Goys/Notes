[1011. 在 D 天内送达包裹的能力](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)

> 传送带上的包裹必须在 `days` 天内从一个港口运送到另一个港口。
>
> 传送带上的第 `i` 个包裹的重量为 `weights[i]`。每一天，我们都会按给出重量（`weights`）的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。
>
> 返回能在 `days` 天内将传送带上的所有包裹送达的船的最低运载能力。

---

懒加载思路，每次二分时计算对应的值。

```go
func shipWithinDays(weights []int, days int) int {
    sum := 0
    maxn := -1
    for _, x := range weights {
        sum += x
        maxn = max(x, maxn)
    }
    day := sort.Search(sum - maxn, func(i int) bool {
        return solve(weights, i + maxn) <= days
    })
    return day + maxn
}

func solve(weights []int, load int) int {
    curLoad := load
    cnt := 1
    for _, x := range weights {
        if curLoad < x {
            curLoad = load
            cnt ++
        }
        curLoad -= x
    }
    return cnt
}
```

