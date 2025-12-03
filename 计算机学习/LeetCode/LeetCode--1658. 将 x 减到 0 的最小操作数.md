[1658. 将 x 减到 0 的最小操作数](https://leetcode.cn/problems/minimum-operations-to-reduce-x-to-zero/)

> 给你一个整数数组 `nums` 和一个整数 `x` 。每一次操作时，你应当移除数组 `nums` 最左边或最右边的元素，然后从 `x` 中减去该元素的值。请注意，需要 **修改** 数组以供接下来的操作使用。
>
> 如果可以将 `x` **恰好** 减到 `0` ，返回 **最小操作数** ；否则，返回 `-1` 。

---

一般这种从两边取数字的都要变换一下思路，思考一下变成从中间取，这样就变成了 `中间的元素 + x == 数组总和` ，这样就便于计算了

```go
func minOperations(nums []int, x int) int {
    sum := 0
    for _, m := range nums {
        sum += m
    }
    if sum < x {
        return -1
    }
    l := 0
    curSum := 0
    ans := 114514
    for r, k := range nums {
        curSum += k
        for curSum + x > sum {
            curSum -= nums[l]
            l ++
        }
        if curSum + x == sum {
            ans = min(ans, len(nums) - (r - l + 1))
        }
    }
    if ans == 114514 {
        return -1
    }
    return ans
}
```

