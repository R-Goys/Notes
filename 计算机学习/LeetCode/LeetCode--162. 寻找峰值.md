[162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)

> 峰值元素是指其值严格大于左右相邻值的元素。
>
> 给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 **任何一个峰值** 所在位置即可。
>
> 你可以假设 `nums[-1] = nums[n] = -∞` 。
>
> 你必须实现时间复杂度为 `O(log n)` 的算法来解决此问题。

---

感觉像这种二分题目，画图是最好想出来的。

```go
func findPeakElement(nums []int) int {
    getNum := func(i int) int {
        if i == -1 || i == len(nums) {
            return math.MinInt64
        }
        return nums[i]
    }

    l, r := 0, len(nums) - 1
    for l <= r {
        mid := (l + r) / 2
        if getNum(mid) < getNum(mid + 1) {
            l = mid + 1
        } else {
            r = mid - 1
        }
    }
    return l
}

```

