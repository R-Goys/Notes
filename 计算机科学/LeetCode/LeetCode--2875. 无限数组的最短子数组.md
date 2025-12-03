[2875. 无限数组的最短子数组](https://leetcode.cn/problems/minimum-size-subarray-in-infinite-array/)

> 给你一个下标从 **0** 开始的数组 `nums` 和一个整数 `target` 。
>
> 下标从 **0** 开始的数组 `infinite_nums` 是通过无限地将 nums 的元素追加到自己之后生成的。
>
> 请你从 `infinite_nums` 中找出满足 **元素和** 等于 `target` 的 **最短** 子数组，并返回该子数组的长度。如果不存在满足条件的子数组，返回 `-1` 。

---

很明显，我们肯定会有超出当前数组总和的 target，所以先取模，然后通过除以当前总和乘上长度得到 baseLen，然后把两个数组拼在一起进行滑动窗口即可：

```go
func minSizeSubarray(nums []int, target int) int {
    sum := 0
    for _, x := range nums {
        sum += x
    }
    baseLen := target / sum * len(nums)
    target %= sum
    l := 0
    curNum := 0
    ans := 0x3f3f3f3f
    nums = append(nums, nums...)
    for r, x := range nums {
        curNum += x
        for curNum > target {
            curNum -= nums[l]
            l ++
        }
        if curNum == target {
            ans = min(r - l + 1, ans)
        }
    }
    if ans >= 0x3f3f3f3f {
        return -1
    }
    return ans + baseLen
}
```

