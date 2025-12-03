[712. 两个字符串的最小ASCII删除和](https://leetcode.cn/problems/minimum-ascii-delete-sum-for-two-strings/)

> 给定两个字符串`s1` 和 `s2`，返回 *使两个字符串相等所需删除字符的 **ASCII** 值的最小和* 。

---

动态规划，此处需要得到删除字符的最小值，看着唬人，其实就是个编辑距离的另一种形式，假设我们s1长度为x，s2长度为y，那么我们的值可以通过上一个状态来获取：`f[x][y] = min(f[x - 1][y] + int(s1[x - 1]), f[x][y - 1] + int(s2[y - 1]))`，这样，通过之前的状态，可以获取当前的状态的值，当然，肯定可以进行滚动优化，这里不过多赘述，直接抛出未优化的版本。

```go
func minimumDeleteSum(s1 string, s2 string) int {
    n, m := len(s1), len(s2)
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
        if i > 0 {
            f[i][0] = f[i - 1][0] + int(s1[i - 1])
        }
    }
    for i := 1; i <= m; i ++ {
        f[0][i] = f[0][i - 1] + int(s2[i - 1])
    }

    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if s1[i - 1] == s2[j - 1] {
                f[i][j] = f[i - 1][j - 1]
            } else {
                f[i][j] = min(f[i - 1][j] + int(s1[i - 1]), f[i][j - 1] + int(s2[j - 1]))
            }
        }
    }
    return f[n][m]
}
```

空间优化版本：

```go
func minimumDeleteSum(s1 string, s2 string) int {
    n, m := len(s1), len(s2)
    f := make([][]int, 2)
    f[0] = make([]int, m + 1)
    f[1] = make([]int, m + 1)

    for i := 1; i <= m; i ++ {
        f[0][i] = f[0][i - 1] + int(s2[i - 1])
    }

    for i := 1; i <= n; i ++ {
        f[i % 2][0] = f[(i - 1) % 2][0] + int(s1[i - 1])
        for j := 1; j <= m; j ++ {
            if s1[i - 1] == s2[j - 1] {
                f[i % 2][j] = f[(i - 1) % 2][j - 1]
            } else {
                f[i % 2][j] = min(f[(i - 1) % 2][j] + int(s1[i - 1]), f[i % 2][j - 1] + int(s2[j - 1]))
            }
        }
    }
    return f[n % 2][m]
}
```

二刷：

还是求公共子序列，不过这次不是求最长，而是 ASCII 码最大的子序列，线性 dp 这些题太像了：

```go
func minimumDeleteSum(s1 string, s2 string) int {
    n, m := len(s1), len(s2)
    sum1, sum2 := 0, 0
    for i := 0 ; i < n; i ++ {
        sum1 += int(s1[i])
    }
    for i := 0 ; i < m; i ++ {
        sum2 += int(s2[i])
    }
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if s1[i - 1] == s2[j - 1] {
                f[i][j] = f[i - 1][j - 1] + int(s1[i - 1])
            } else {
                f[i][j] = max(f[i - 1][j], f[i][j - 1])
            }
        }
    }
    return sum1 + sum2 - 2 * f[n][m]
}
```

