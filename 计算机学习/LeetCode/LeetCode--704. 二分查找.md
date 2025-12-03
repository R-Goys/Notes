[704. 二分查找](https://leetcode.cn/problems/binary-search/)

> 给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target` ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。

---

模板题目，无需多言

```go
func search(nums []int, target int) int {
    l, r := 0, len(nums) - 1
    for l <= r {
        mid := (l + r) / 2
        if nums[mid] > target {
            r = mid - 1
        } else if nums[mid] < target {
            l = mid + 1
        } else {
            return mid
        }
    }
    return -1
}
```

for循环那里取等号，是为了方便判断区间长度为1的时候，能够正确判断