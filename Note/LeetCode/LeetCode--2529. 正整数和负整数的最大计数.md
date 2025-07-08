[2529. 正整数和负整数的最大计数](https://leetcode.cn/problems/maximum-count-of-positive-integer-and-negative-integer/)

> 给你一个按 **非递减顺序** 排列的数组 `nums` ，返回正整数数目和负整数数目中的最大值。
>
> - 换句话讲，如果 `nums` 中正整数的数目是 `pos` ，而负整数的数目是 `neg` ，返回 `pos` 和 `neg`二者中的最大值。
>
> **注意：**`0` 既不是正整数也不是负整数。

---

二分查找，寻找 0 和 1 的位置，就可以分割左右边界。

```go
func maximumCount(nums []int) int {
    neg := bio(nums, 0)
    pos := bio(nums, 1)
    return max(neg, len(nums) - pos)    
}

func bio(nums []int, target int) int {
    l, r := 0, len(nums)
    for l < r {
        mid := (l + r) / 2
        if nums[mid] < target {
            l = mid + 1
        } else {
            r = mid
        }
    }
    return l
}
```

