[120. 三角形最小路径和](https://leetcode.cn/problems/triangle/)

> 给定一个三角形 `triangle` ，找出自顶向下的最小路径和。
>
> 每一步只能移动到下一行中相邻的结点上。**相邻的结点** 在这里指的是 **下标** 与 **上一层结点下标** 相同或者等于 **上一层结点下标 + 1** 的两个结点。也就是说，如果正位于当前行的下标 `i` ，那么下一步可以移动到下一行的下标 `i` 或 `i + 1` 。

---

去年写过，忘咋写了，直接写了个暴力动态规划，先将边界部分初始化完，再将无需考虑边界条件的计算出来：

```c
func minimumTotal(triangle [][]int) int {
    n := len(triangle)
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, i + 1)
    }
    f[0][0] = triangle[0][0]
    for i := 1; i < n; i ++ {
        f[i][0] = f[i - 1][0] + triangle[i][0]
        f[i][i] = f[i - 1][i - 1] + triangle[i][i]
    }
    for i := 1; i < n; i ++ {
        for j := 1; j < len(triangle[i]) - 1; j ++ {
            f[i][j] = min(f[i - 1][j], f[i - 1][j - 1]) + triangle[i][j]
        }
    }
    ans := 999999
    for i := 0; i < len(triangle[n - 1]); i ++ {
        ans = min(f[n - 1][i], ans)
    }
    return ans
}
```

优化：

```go
func minimumTotal(triangle [][]int) int {
    n := len(triangle)
    f := make([]int, n)
    f[0] = triangle[0][0]
    for i := 1; i < n; i ++ {
        f[i] = f[i - 1] + triangle[i][i]
        for j := i - 1; j > 0; j -- {
            f[j] = min(f[j - 1], f[j]) + triangle[i][j]
        }
        f[0] += triangle[i][0]
    }
    ans := math.MaxInt32
    for i := 0; i < n; i++ {
        ans = min(ans, f[i])
    }
    return ans
}
```

在进行空间优化的时候，需要注意如果建立一维数组，需要对左右两侧的边界进行特殊处理。

二刷：

```go
func minimumTotal(triangle [][]int) int {
    n := len(triangle)
    ans := 0x3f3f3f3f
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, i + 1)
    }
    f[0][0] = triangle[0][0]
    for i := 1; i < n; i ++ {
        f[i][0] = f[i - 1][0] + triangle[i][0]
        f[i][i] = f[i - 1][i - 1] + triangle[i][i]
    }
    for i := 1; i < n; i ++ {
        for j := 1; j < len(triangle[i]) - 1; j ++ {
            f[i][j] = min(f[i - 1][j], f[i - 1][j - 1]) + triangle[i][j]
        }
    }
    for i := 0; i < len(triangle[n - 1]); i ++ {
        ans = min(f[n - 1][i], ans)
    }
    return ans
}
```

空间优化

```go
func minimumTotal(triangle [][]int) int {
    n := len(triangle)
    ans := 0x3f3f3f3f
    f := make([]int, n)
    f[0] = triangle[0][0]
    for i := 1; i < n; i ++ {
        f[i] = f[i - 1] + triangle[i][i]
        for j := i - 1; j >= 1; j -- {
            f[j] = min(f[j], f[j - 1]) + triangle[i][j]
        }
        f[0] += triangle[i][0]
    }
    for i := 0; i < len(triangle[n - 1]); i ++ {
        ans = min(f[i], ans)
    }
    return ans
}
```

