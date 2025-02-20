[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

> 给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。

---

## 正文

创建一个visited二维布尔切片，来判断当前格子走没走过，遍历grid数组，发现'1'，就开始感染，同时岛屿总数+1，此后若再遇见'1'，并且没有被遍历过，说明该陆地没有与之前的岛屿相连，即没有被感染，岛屿数量直接+1即可。



```go
func numIslands(grid [][]byte) int {
    n := len(grid)
    m := len(grid[0])
    num := 0
    visited := make([][]bool, n)
    for i := 0; i < n; i++ {
        visited[i] = make([]bool, m)
    }
    for i := 0; i < n; i++ {
        for j := 0; j < m; j ++{ 
            if visited[i][j] {
                continue
            }
            if grid[i][j] == '1' {
                num ++
                infect(&grid, i, j, &visited)
            }
        }
    }
    return num
}

func infect(grid *[][]byte, i int, j int, visited *[][]bool)  {
  	if i < 0 || i >= len(*grid) || j < 0 || j >= len((*grid)[0]) || (*visited)[i][j] || (*grid)[i][j] == '0' {
        //超出了界限，或者已经遍历过，或是为海洋部分，直接return
        return
    }
    if (*grid)[i][j] == '1' {
        //是陆地，设为true
        (*visited)[i][j] = true
    }
	//xy偏移量
    dx := []int{1, 0, 0, -1}
    dy := []int{0, 1, -1, 0}

    for k := 0; k < 4; k ++ {
        infect(grid, i + dx[k], j + dy[k], visited)
    }
}
```

----



