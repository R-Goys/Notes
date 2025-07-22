[853. 车队](https://leetcode.cn/problems/car-fleet/)

> 在一条单行道上，有 `n` 辆车开往同一目的地。目的地是几英里以外的 `target` 。
>
> 给定两个整数数组 `position` 和 `speed` ，长度都是 `n` ，其中 `position[i]` 是第 `i` 辆车的位置， `speed[i]` 是第 `i` 辆车的速度(单位是英里/小时)。
>
> 一辆车永远不会超过前面的另一辆车，但它可以追上去，并以较慢车的速度在另一辆车旁边行驶。
>
> **车队** 是指并排行驶的一辆或几辆汽车。车队的速度是车队中 **最慢** 的车的速度。
>
> 即便一辆车在 `target` 才赶上了一个车队，它们仍然会被视作是同一个车队。
>
> 返回到达目的地的车队数量 。

---

按照距离远近进行排序，然后根据速度进行单调栈，满足特定条件的子序列就是一个车队。

```go
func carFleet(target int, position []int, speed []int) int {
    car := make([][2]int, len(position))
    for i := range position {
		car[i][0] = position[i]
		car[i][1] = speed[i]
    }
	sort.Slice(car, func(i, j int) bool {
		return car[i][0] < car[j][0]
	})
    src := make([]float64, len(position))
    for i, x := range car {
        src[i] = float64(target - x[0]) / float64(x[1])
    }
    ans := 0
    maxn := 0.0
	for i := len(src) - 1; i >= 0; i -- {
		if src[i] > maxn {
			maxn = src[i]
			ans ++
		}
	}
    return ans
}
```

