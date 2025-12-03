[45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)

> 给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。
>
> 每个元素 `nums[i]` 表示从索引 `i` 向后跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处:
>
> - `0 <= j <= nums[i]` 
> - `i + j < n`
>
> 返回到达 `nums[n - 1]` 的最小跳跃次数。生成的测试用例可以到达 `nums[n - 1]`。

---

跳跃游戏的变种，稍微要多处理一下，比较清晰易懂，但是时间复杂度高的。

```go
func jump(nums []int) int {
    idx := 0
    for i := 0; i < len(nums) - 1; {
        MaxStep := 0
        p := 0
        for j := 1; j <= nums[i]; j ++ {
            if i + j >= len(nums) - 1 {
                return idx + 1
            }
            if MaxStep < nums[i + j] {
                MaxStep = nums[i + j]
                p = j
            }
            MaxStep--
        }
        idx++
        i += p
    }
    return idx
}
```

优化版，时间复杂度O(n)，end代表第一次跳的最远距离，而MaxStep代表下一次跳的最远距离，根据这两个参数，就可以理解这个step的数量是如何计算的了。

```go
func jump(nums []int) int {
    n := len(nums)
    MaxStep := 0
    end := 0
    cnt := 0
    for i := 0; i < n - 1; i ++ {
        MaxStep = max(MaxStep, i + nums[i])
        if end == i {
            end = MaxStep
            cnt ++
        }
    }
    return cnt
}
```

----

二刷

```go
func jump(nums []int) int {
    i, ans := 0, 0
    for i < len(nums) - 1 {
        maxStep, offset := 0, 0
        for j := 1; j <= nums[i]; j ++ {
            if i + j >= len(nums) - 1 {
                return ans + 1
            }
            if maxStep < nums[i + j] {
                maxStep = nums[i + j]
                offset = j
            }
            //这里是为了保证每次步数正常更新
            maxStep --
        }
        i += offset
        ans ++
    }
    return ans
}
```

另一种方法：

```go
func jump(nums []int) int {
    end := 0
    ans := 0
    MaxStep := 0
    //注意，到达最后一个位置的时候就可以结束循环了。
    for i := 0; i < len(nums) - 1; i ++ {
        MaxStep = max(MaxStep, nums[i] + i)
        if end == i {
            end = MaxStep
            ans ++
        }
    }
    return ans
}
```

