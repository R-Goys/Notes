[2070. 每一个查询的最大美丽值](https://leetcode.cn/problems/most-beautiful-item-for-each-query/)

> 给你一个二维整数数组 `items` ，其中 `items[i] = [pricei, beautyi]` 分别表示每一个物品的 **价格** 和 **美丽值** 。
>
> 同时给你一个下标从 **0** 开始的整数数组 `queries` 。对于每个查询 `queries[j]` ，你想求出价格小于等于 `queries[j]` 的物品中，**最大的美丽值** 是多少。如果不存在符合条件的物品，那么查询的结果为 `0` 。
>
> 请你返回一个长度与 `queries` 相同的数组 `answer`，其中 `answer[j]`是第 `j` 个查询的答案。

---

排序 + 二分，先将 items 按照分数排序，然后统计前缀最大值，随后进行二分即可。

```go
func maximumBeauty(items [][]int, queries []int) []int {
    slices.SortFunc(items, func(a, b []int) int {
        return a[0] - b[0]
    })
    ans := []int{}
    pref := make([]int, 0)
    pre := -1
    for _, x := range items {
        pre = max(x[1], pre)
        pref = append(pref, pre)
    }
    
    for _, x := range queries {
        idx := sort.Search(len(items), func(i int) bool {
            return items[i][0] > x
        })
        if idx > 0 {
            ans = append(ans, pref[idx - 1])
        } else {
            ans = append(ans, 0)
        }
    }
    return ans
}
```

