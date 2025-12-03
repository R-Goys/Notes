[475. 供暖器](https://leetcode.cn/problems/heaters/)

> 冬季已经来临。 你的任务是设计一个有固定加热半径的供暖器向所有房屋供暖。
>
> 在加热器的加热半径范围内的每个房屋都可以获得供暖。
>
> 现在，给出位于一条水平线上的房屋 `houses` 和供暖器 `heaters` 的位置，请你找出并返回可以覆盖所有房屋的最小加热半径。
>
> **注意**：所有供暖器 `heaters` 都遵循你的半径标准，加热的半径也一样。

---

o(nlogn) 的复杂度，瓶颈为排序，双指针做法。

```go
func findRadius(houses []int, heaters []int) int {
    sort.Ints(houses)
    sort.Ints(heaters)
    ans := 0
    j := 0
    for _, h := range houses {
        for j + 1 < len(heaters) && abs(h - heaters[j]) >= abs(h - heaters[j + 1]) {
            j ++
        }
        d := abs(h - heaters[j])
        ans = max(ans, d)
    }
    return ans
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

