[740. 删除并获得点数](https://leetcode.cn/problems/delete-and-earn/)

> 给你一个整数数组 `nums` ，你可以对它进行一些操作。
>
> 每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除 **所有** 等于 `nums[i] - 1` 和 `nums[i] + 1` 的元素。
>
> 开始你拥有 `0` 个点数。返回你能通过这些操作获得的最大点数。

---

这道题的主要问题在于如何删除我们的前一个和后一个节点，但是这里我们的最终结果可以简化为不选择前一个节点即可，因为对于后面的元素来说，不选择前一个元素，和前一个元素不选择后面的元素是一样的意思，其实就和打家劫舍这一道题是一个思路。

我们可以将所有的数字化成数组，key表示数字，val表示这个数的所有的val的和，随后进行dp

```go
func deleteAndEarn(nums []int) int {
    maxVal := 0
    for _, val := range nums {
        maxVal = max(maxVal, val)
    }
    count := make([]int, maxVal+1)
    for _, val := range nums {
        count[val] += val
    }
    return dp(count)
}

func dp(nums []int) int {
    first, second := nums[0], max(nums[0], nums[1])
    for i := 2; i < len(nums); i++ {
        first, second = second, max(first+nums[i], second)
    }
    return second
}
```

