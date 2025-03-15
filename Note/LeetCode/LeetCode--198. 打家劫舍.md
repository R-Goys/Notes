[198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

----

像这种dp都是三分钟出来了，难的又做不出来，感觉目前就是纯记忆了，真得废了

```go
func rob(nums []int) int {
    n := len(nums)
    f := make([][]int, n)

    for i := 0; i < n; i ++ {
        f[i] = make([]int, 2)
    }
    f[0][1] = nums[0]

    for i := 1; i < n; i ++ {
        f[i][0] = max(f[i - 1][1], f[i - 1][0])
        f[i][1] = f[i - 1][0] + nums[i]
    }

    return max(f[n - 1][0], f[n - 1][1])
}
```

**滚动数组优化**

byd，这几把dp空间复杂度都O(1)了，好虾仁

```go
func rob(nums []int) int {
    n := len(nums)
    f := make([][]int, 2)

    for i := 0; i < 2; i ++ {
        f[i] = make([]int, 2)
    }
    f[0][1] = nums[0]

    for i := 1; i < n; i ++ {
        f[i%2][0] = max(f[(i - 1)%2][1], f[(i - 1)%2][0])
        f[i%2][1] = f[(i - 1)%2][0] + nums[i]
    }

    return max(f[(n - 1)%2][0], f[(n - 1)%2][1])
}
```

