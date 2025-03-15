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

**无需二维数组的做法**

虽然这种看起来还算优雅吧，但是在计算路径的时候可能不是太友好。

```go
func rob(nums []int) int {
    n := len(nums)
    if n == 0 {
        return 0
    }
    if n == 1 {
        return nums[0]
    }
    f := make([]int, n + 2)

    nums[1] = max(nums[1], nums[0])
    for i := 2; i < n + 2; i ++ {
        f[i] = max(f[i - 1], f[i - 2] + nums[i - 2])
    }

    return f[n + 1]
}
```

**输出路径**

这部分还是花了一点时间，总体思路是用三位数组解决的，至于测试，我是通过遍历path，将ans加起来，然后return，通过了力扣的测试，应该是没啥问题的。

每次走都有一次路径的保存，同时有偷和不偷的两种选择，这个思路和上面是一样的，虽然这里完全可以优化很多空间，但是为了好理解，我直接把没有优化过的放出来了。

```go
package main

import "fmt"

func rob(nums []int) int {
	n := len(nums)
	f := make([][]int, n)
	path := make([][][]int, n)
	for i := 0; i < n; i++ {
		f[i] = make([]int, 2)
		path[i] = make([][]int, 2)
	}
	f[0][1] = nums[0]
	path[0][1] = append([]int{}, nums[0])

	for i := 1; i < n; i++ {
		if f[i-1][0] > f[i-1][1] {
			f[i][0] = f[i-1][0]
			path[i][0] = append([]int{}, path[i-1][0]...)
		} else {
			f[i][0] = f[i-1][1]
			path[i][0] = append([]int{}, path[i-1][1]...)
		}

		f[i][1] = f[i-1][0] + nums[i]
		cur := append([]int{}, path[i-1][0]...)
		cur = append(cur, nums[i])

		path[i][1] = cur
	}
	if f[n-1][0] > f[n-1][1] {
		fmt.Println(path[n-1][0])
		return f[n-1][0]
	} else {
		fmt.Println(path[n-1][1])
		return f[n-1][1]
	}
}

func main() {
	fmt.Println(rob([]int{1, 1, 1, 2}))
}

```



