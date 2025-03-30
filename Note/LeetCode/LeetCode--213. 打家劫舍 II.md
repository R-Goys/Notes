[213. 打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)

> 你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 **围成一圈** ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警** 。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 **在不触动警报装置的情况下** ，今晚能够偷窃到的最高金额。

----

在打家劫舍I中，我们用了二维数组来记录两种情况，由于这里加入了环形的条件，所以我们可以进一步划分为四种情况，依旧使用滚动数组，空间O(1)

```go
func rob(nums []int) int {
        n := len(nums)
        f := make([][]int, 4)
        f[0], f[1], f[2], f[3] = make([]int, 2), make([]int, 2), make([]int, 2), make([]int, 2)
        f[1][0] = nums[0]
        for i := 1; i < n; i ++ {
                f[0][i % 2] = max(f[1][(i - 1)%2], f[0][(i - 1)%2])
                if i != n - 1 {
                        f[1][i % 2] = max(f[0][(i - 1) % 2] + nums[i], f[1][(i - 1) % 2])
                } else {
                        f[1][i % 2] = max(f[1][(i - 1) % 2], f[0][(i - 1) % 2])
                }
                f[2][i % 2] = max(f[3][(i - 1) % 2], f[2][(i - 1) % 2])
                f[3][i % 2] = max(f[2][(i - 1) % 2] + nums[i], f[3][(i - 1) % 2])
        }
        return max(f[0][(n - 1) % 2], f[1][(n - 1) % 2], f[2][(n - 1) % 2], f[3][(n - 1) % 2])
}
```

另一种方法也是分开处理，将左右分为两个数组分别进行处理，也是O(1)的空间复杂度。

```go
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    if n == 2 {
        return max(nums[0], nums[1])
    }
    return max(_rob(nums[:n-1]), _rob(nums[1:]))
}

func _rob(nums []int) int {
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

