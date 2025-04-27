[474. 一和零](https://leetcode.cn/problems/ones-and-zeroes/)

> 给你一个二进制字符串数组 `strs` 和两个整数 `m` 和 `n` 。
>
> 请你找出并返回 `strs` 的最大子集的长度，该子集中 **最多** 有 `m` 个 `0` 和 `n` 个 `1` 。
>
> 如果 `x` 的所有元素也是 `y` 的元素，集合 `x` 是集合 `y` 的 **子集** 。

---

首先明确我们要干什么：选择子集合，保证 0 和 1 的数量分别小于等于 m 和 n ，同时保证子集长度最大。

这样就包含了三种状态，跟洛谷上面有一道糖果的题很像，我们需要开辟三维数组来进行dp。

选择前 i 个元素，保证子集中包含的 0 和 1 分别小于 j 和 k ，其实就是背包问题：

```go
func findMaxForm(strs []string, m int, n int) int {
    length := len(strs)
    f := make([][][]int, length + 1)
    for i := 0; i <= length; i ++ {
        f[i] = make([][]int, m + 1)
        for j := 0; j <= m; j ++ {
            f[i][j] = make([]int, n + 1)
        }
    }

    for i := 0; i < length; i ++ {
        zeroCnt := strings.Count(strs[i], "0")
        oneCnt := len(strs[i]) - zeroCnt
        for j := 0; j <= m; j ++ {
            for k := 0; k <= n; k ++ {
                f[i + 1][j][k] = f[i][j][k]
                if j >= zeroCnt && k >= oneCnt {
                    f[i + 1][j][k] = max(f[i + 1][j][k], f[i][j - zeroCnt][k - oneCnt] + 1)
                }
            }
        }
    }
    return f[length][m][n]
}
```

