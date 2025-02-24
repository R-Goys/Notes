[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

> 以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

---

## 排序 + 遍历

先按左端点排序切片，随后用变量l和r维护左右端点，当遍历intervals的时候，如果发现当前的r大于l，此时可以进行区间合并，直接`intervals[i][1]`和`r`取最大值即可

```go
func merge(intervals [][]int) [][]int {
	sort.Slice(intervals,func(i, j int) bool {
		return intervals[i][0] < intervals[j][0]
	})
    l := intervals[0][0]
    r := intervals[0][1]
    n := len(intervals)
    var ans [][]int
    for i := 0; i < n; i ++ {
        if r >= intervals[i][0] {
            r = max(r, intervals[i][1])
            continue
        }
        ans = append(ans, []int{l, r})
        l = intervals[i][0]
        r = intervals[i][1]
    }
    ans = append(ans, []int{l, r})
    return ans
}
```

