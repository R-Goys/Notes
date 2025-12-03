[1182. 与目标颜色间的最短距离](https://leetcode.cn/problems/shortest-distance-to-target-color/)

> 给你一个数组 `colors`，里面有 `1`、`2`、 `3` 三种颜色。
>
> 我们需要在 `colors` 上进行一些查询操作 `queries`，其中每个待查项都由两个整数 `i` 和 `c` 组成。
>
> 现在请你帮忙设计一个算法，查找从索引 `i` 到具有目标颜色 `c` 的元素之间的最短距离。
>
> 如果不存在解决方案，请返回 `-1`。

---

哈希表存 1，2，3 的下标，然后遍历 queries 二分

```go
func shortestDistanceColor(colors []int, queries [][]int) []int {
    src := [4][]int{}
    src[1] = append(src[1], -1)
    src[2] = append(src[2], -1)
    src[3] = append(src[3], -1)
    for i, x := range colors {
        src[x] = append(src[x], i)
    }
    ans := []int{}
    for _, x := range queries {
        idx := sort.Search(len(src[x[1]]) - 1, func(i int) bool {
            return src[x[1]][i] >= x[0]
        })
        if idx == 0 {
            ans = append(ans, -1)
        } else {
            ans = append(ans, min(abs(src[x[1]][idx] - x[0]), abs(src[x[1]][max(1, idx - 1)] - x[0])))
        }
    }
    return ans
}

func abs(x int) int {
    if x > 0 {
        return x
    }
    return -x
}
```

