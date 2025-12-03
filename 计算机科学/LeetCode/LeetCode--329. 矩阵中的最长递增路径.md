[329. 矩阵中的最长递增路径](https://leetcode.cn/problems/longest-increasing-path-in-a-matrix/)

> 给定一个 `m x n` 整数矩阵 `matrix` ，找出其中 **最长递增路径** 的长度。
>
> 对于每个单元格，你可以往上，下，左，右四个方向移动。 你 **不能** 在 **对角线** 方向上移动或移动到 **边界外**（即不允许环绕）。

---

记忆化搜索，关于为什么mem可以被复用，实际上，mem是指这个地方之后的单调递增路径，所以是不会冲突的。

```go
func longestIncreasingPath(matrix [][]int) int {
    n, m := len(matrix), len(matrix[0])
    ans := 0
    mem := make([][]int, n)
    for i := 0; i < n; i ++ {
        mem[i] = make([]int, m)
    }
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            ans = max(ans, search(matrix, mem, i, j))
        }
    }
    return ans
}

func search(matrix, mem [][]int, i, j int) int {
    if mem[i][j] != 0 {
        return mem[i][j]
    }
    biggest := 1
    dx := []int{1, 0, -1, 0}
    dy := []int{0, 1, 0, -1}
    for k := 0; k < 4; k ++ {
        x := dx[k] + i
        y := dy[k] + j
        if x < 0 || x >= len(matrix) || y < 0 || y >= len(matrix[0]) {
            continue
        }
        if matrix[x][y] > matrix[i][j] {
            biggest = max(biggest, 1 + search(matrix, mem, x, y))
        }
    }
    mem[i][j] = biggest
    return biggest
}
```

二刷，使用拓扑序列的方法，首先计算出度，然后将出度为0，也就是我们的递增序列路径的终点加入队列中，然后反向地去搜索路径，每次找到一个点，便将他的出度减一，直到它的出度为 0 ，则可以加入队列之中，这样则可以保证我们能够只遍历一次这一个点。

```go
var dirs = [][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}

func longestIncreasingPath(matrix [][]int) int {
    n, m := len(matrix), len(matrix[0])
    outDegree := make([][]int, n)
    for i := 0; i < n; i ++ {
        outDegree[i] = make([]int, m)
    }
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            for _, dir := range dirs {
                neI := i + dir[0]
                neJ := j + dir[1]
                if neI >= 0 && neJ >= 0 && neI < n && neJ < m && matrix[i][j] < matrix[neI][neJ] {
                    outDegree[i][j] ++
                }
            }
        }
    }
    q := [][2]int{}
    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            if outDegree[i][j] == 0 {
                q = append(q, [2]int{i, j})
            }
        }
    }
    ans := 0
    for len(q) != 0 {
        ans ++
        for _, pos := range q {
            q = q[1:]
            i, j := pos[0], pos[1]
            for _, dir := range dirs {
                preI := i + dir[0]
                preJ := j + dir[1]
                if preI >= 0 && preJ >= 0 && preI < n && preJ < m && matrix[preI][preJ] < matrix[i][j] {
                    outDegree[preI][preJ] --
                    if outDegree[preI][preJ] == 0 {
                        q = append(q, [2]int{preI, preJ})
                    }
                }
            }
        }
    }
    return ans
}
```

