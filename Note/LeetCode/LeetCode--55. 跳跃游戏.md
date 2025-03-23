[55. 跳跃游戏](https://leetcode.cn/problems/jump-game/)

> 给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

---

贪心，记录最右边界即可

```go
func canJump(nums []int) bool {
    n := len(nums)
    MaxStep := 0
    for i := 0; i < n; i ++ {
        if i <= MaxStep {
            MaxStep = max(MaxStep, i + nums[i])
            if MaxStep >= n - 1 {
                return true
            }
        }
    }
    return false
}
```

