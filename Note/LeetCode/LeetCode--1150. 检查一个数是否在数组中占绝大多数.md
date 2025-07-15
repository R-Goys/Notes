[1150. 检查一个数是否在数组中占绝大多数](https://leetcode.cn/problems/check-if-a-number-is-majority-element-in-a-sorted-array/)

> 给出一个按 **非递减** 顺序排列的数组 `nums`，和一个目标数值 `target`。假如数组 `nums` 中绝大多数元素的数值都等于 `target`，则返回 `True`，否则请返回 `False`。
>
> 所谓占绝大多数，是指在长度为 `N` 的数组中出现必须 **超过 `N/2`** **次**。

---

对 target 二分查找左右边界即可。

```go
func isMajorityElement(nums []int, target int) bool {
    l := sort.SearchInts(nums, target)
    r := sort.SearchInts(nums, target + 1)
    if l >= len(nums) || nums[l] != target {
        return false
    }
    return (r - l) > len(nums) / 2
}
```

