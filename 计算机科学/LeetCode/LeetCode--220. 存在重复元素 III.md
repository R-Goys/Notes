[220. 存在重复元素 III](https://leetcode.cn/problems/contains-duplicate-iii/)

> 给你一个整数数组 `nums` 和两个整数 `indexDiff` 和 `valueDiff` 。
>
> 找出满足下述条件的下标对 `(i, j)`：
>
> - `i != j`,
> - `abs(i - j) <= indexDiff`
> - `abs(nums[i] - nums[j]) <= valueDiff`
>
> 如果存在，返回 `true` *；*否则，返回 `false` 。

---

桶排，以前没用过，现在长见识了。

```go
func containsNearbyAlmostDuplicate(nums []int, k int, t int) bool {
    n := len(nums)
    mapBuckets := make(map[int]int)      
    size := t + 1
    for i := 0; i < n; i ++ {
        u := nums[i]
        idx := getIdx(u, size)

        if _, exists := mapBuckets[idx]; exists {
            return true
        }

        l, r := idx - 1, idx + 1
        if val, exists := mapBuckets[l]; exists && abs(u - val) <= t {
            return true
        }
        if val, exists := mapBuckets[r]; exists && abs(u - val) <= t {
            return true
        }

        mapBuckets[idx] = u

        if i >= k {
            // 这里可以直接删除的原因是，如果桶中有元素是存在当前区间的，
            // 说明我们应该早就返回 true 了，但是没有，所以我们当前区间没有
            // 对应的数值，所以可以直接删除，不影响。
            removeIdx := getIdx(nums[i - k], size)
            delete(mapBuckets, removeIdx)
        }
    }
    return false
}

func getIdx(u int, size int) int {
    if u >= 0 {
        return u / size
    }
    return (u + 1) / size - 1
}

func abs(x int) int {
    if x > 0 {
        return x
    }
    return -x
}
```

