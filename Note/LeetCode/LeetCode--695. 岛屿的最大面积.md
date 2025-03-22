[695. 岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)

> 给你一个大小为 `m x n` 的二进制矩阵 `grid` 。
>
> **岛屿** 是由一些相邻的 `1` (代表土地) 构成的组合，这里的「相邻」要求两个 `1` 必须在 **水平或者竖直的四个方向上** 相邻。你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。
>
> 岛屿的面积是岛上值为 `1` 的单元格的数目。
>
> 计算并返回 `grid` 中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。

老样子，没啥技巧，dfs递归遍历，需要维护当前岛屿面积最大值和当前岛屿大小，和之前岛屿数量那道题差不多。

```go
var dx = []int{1, 0, -1, 0}
var dy = []int{0, 1, 0, -1}

func maxAreaOfIsland(grid [][]int) int {
    n, m := len(grid), len(grid[0])
    visited := make([][]bool, n)
    for i := 0; i < n; i ++ {
        visited[i] = make([]bool, m)
    }
    sum := 0
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            cur := 0
            infect(i, j, grid, visited, &sum, &cur)
        }
    }
    return sum
}

func infect(x, y int, grid [][]int, visited [][]bool, sum, cur *int) {
    (*sum) = max((*sum), (*cur))
    if x < 0 || x >= len(grid) || y < 0 || y >= len(grid[0]) || visited[x][y] == true || grid[x][y] == 0 {
        return
    }
    *cur ++
    visited[x][y] = true
    for i := 0; i < 4; i ++ {
        infect(x + dx[i], y + dy[i], grid, visited, sum, cur)
    }
}
```

