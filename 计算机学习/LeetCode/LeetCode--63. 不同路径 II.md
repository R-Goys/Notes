[63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

> 给定一个 `m x n` 的整数数组 `grid`。一个机器人初始位于 **左上角**（即 `grid[0][0]`）。机器人尝试移动到 **右下角**（即 `grid[m - 1][n - 1]`）。机器人每次只能向下或者向右移动一步。
>
> 网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。机器人的移动路径中不能包含 **任何** 有障碍物的方格。
>
> 返回机器人能够到达右下角的不同路径数量。
>
> 测试用例保证答案小于等于 `2 * 109`。

---

做过过河卒，这题就是降维打击，初始化边界，其实没啥必要这么麻烦，多开辟一层空间就可以省去这么长的初始化了。

```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    n := len(obstacleGrid)
    m := len(obstacleGrid[0])
    f := make([][]int, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]int, m)
    }
    if obstacleGrid[0][0] != 1 {
        for i := 0; i < n; i ++ {
            if obstacleGrid[i][0] == 1 {
                break
            }
            f[i][0] = 1
        }
        for j := 0; j < m; j ++ {
            if obstacleGrid[0][j] == 1 {
                break
            }
            f[0][j] = 1
        }
    }
    for i := 1; i < n; i ++ {
        for j := 1; j < m; j ++ {
            if obstacleGrid[i][j] != 1 {
                f[i][j] = f[i][j - 1] + f[i - 1][j]
            }
        }
    }
    return f[n - 1][m - 1]
}
```

滚动数组优化成一维的：

```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    n := len(obstacleGrid)
    m := len(obstacleGrid[0])
    f := make([]int, m)
    if obstacleGrid[0][0] == 0 {
        f[0] = 1
    }
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            if obstacleGrid[i][j] == 1 {
                f[j] = 0
            }
            if j - 1 >= 0 && obstacleGrid[i][j] == 0 {
                f[j] += f[j - 1]
            }
        }
    }
    return f[m - 1]
}
```

感觉做的dp题型都太像了，感觉像被困在舒适区里面一样。

三刷

```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    n, m := len(obstacleGrid), len(obstacleGrid[0])
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    if obstacleGrid[0][0] == 1 {
        return 0
    }
    f[1][1] = 1
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if obstacleGrid[i - 1][j - 1] == 1 {
                f[i][j] = 0
            } else {
                f[i][j] += f[i - 1][j] + f[i][j - 1]
            }
        }
    }
    return f[n][m]
}
```

