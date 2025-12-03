[3488. 距离最小相等元素查询](https://leetcode.cn/problems/closest-equal-element-queries/)

> 给你一个 **环形** 数组 `nums` 和一个数组 `queries` 。
>
> 对于每个查询 `i` ，你需要找到以下内容：
>
> - 数组 `nums` 中下标 `queries[i]` 处的元素与 **任意** 其他下标 `j`（满足 `nums[j] == nums[queries[i]]`）之间的 **最小** 距离。如果不存在这样的下标 `j`，则该查询的结果为 `-1` 。
>
> 返回一个数组 `answer`，其大小与 `queries` 相同，其中 `answer[i]` 表示查询`i`的结果。

---

将每个数字作为下标，将他们对应的下标存入哈希表的值中。

然后二分查找就行了

```go
func solveQueries(nums []int, queries []int) []int {
    mp := make(map[int][]int, 0)
    for i, x := range nums {
        mp[x] = append(mp[x], i)
    }

    n := len(nums)
    for x, p := range mp {
        i0 := p[0]
        p = slices.Insert(p, 0, p[len(p) - 1] - n)
        mp[x] = append(p, i0 + n)
    }

    for i, q := range queries {
        p := mp[nums[q]]
        if len(p) == 3 {
            queries[i] = -1
        } else {
            j := sort.SearchInts(p, q)
            queries[i] = min(q - p[j - 1], p[j + 1] - q)
        }
    }
    return queries
}
```

