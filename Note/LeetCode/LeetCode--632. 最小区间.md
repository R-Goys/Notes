[632. 最小区间](https://leetcode.cn/problems/smallest-range-covering-elements-from-k-lists/)

> 你有 `k` 个 **非递减排列** 的整数列表。找到一个 **最小** 区间，使得 `k` 个列表中的每个列表至少有一个数包含在其中。
>
> 我们定义如果 `b-a < d-c` 或者在 `b-a == d-c` 时 `a < c`，则区间 `[a,b]` 比 `[c,d]` 小。

---

排序+滑窗，原本想用哈希表对应每个元素的对应的数组索引，但是可能会导致不同数组中有相同元素造成混乱，于是改用二维数组存储对应的关系，随后按照元素值对数组进行排序，最后进行最小值滑窗即可：

```go
func smallestRange(nums [][]int) []int {
    pairs := make([][2]int, 0)
    for i, arr := range nums {
        for _, x := range arr {
            pairs = append(pairs, [2]int{x, i})
        }
    }
    slices.SortFunc(pairs, func(a, b [2]int) int {return a[0] - b[0]})
    l := 0
    empty := len(nums)
    rans, lans := pairs[len(pairs)-1][0], pairs[0][0]
    cnt := make([]int, empty)
    for _, p := range pairs {
        rv, i := p[0], p[1]
        if cnt[i] == 0 {
            empty --
        }
        cnt[i] ++
        for empty == 0 {
            lv, j := pairs[l][0], pairs[l][1]
            if rv - lv < rans - lans {
                rans, lans = rv, lv
            }
            cnt[j] --
            if cnt[j] == 0 {
                empty ++
            }
            l ++
        }
    }
    return []int{lans, rans}
}
```

