[2267. 检查是否有合法括号字符串路径](https://leetcode.cn/problems/check-if-there-is-a-valid-parentheses-string-path/)

> 一个括号字符串是一个 **非空** 且只包含 `'('` 和 `')'` 的字符串。如果下面 **任意** 条件为 **真** ，那么这个括号字符串就是 **合法的** 。
>
> - 字符串是 `()` 。
> - 字符串可以表示为 `AB`（`A` 连接 `B`），`A` 和 `B` 都是合法括号序列。
> - 字符串可以表示为 `(A)` ，其中 `A` 是合法括号序列。
>
> 给你一个 `m x n` 的括号网格图矩阵 `grid` 。网格图中一个 **合法括号路径** 是满足以下所有条件的一条路径：
>
> - 路径开始于左上角格子 `(0, 0)` 。
> - 路径结束于右下角格子 `(m - 1, n - 1)` 。
> - 路径每次只会向 **下** 或者向 **右** 移动。
> - 路径经过的格子组成的括号字符串是 **合法** 的。
>
> 如果网格图中存在一条 **合法括号路径** ，请返回 `true` ，否则返回 `false` 。

---

最近全是 hard ，感觉状态有点差，这道题主要是也是使用 dfs + 记忆化搜索，开辟三维数组，最后一维代表着当前还没有匹配的左括号，然后进行 dfs ，当且仅当坐标为右下角，并且此时还没有匹配的左括号只有一个，且当前位置的括号为右括号，才能够返回 true，所以需要递归传递结果，搜索过程中，也需要检查中途的状态是否正确，比如未匹配的右括号小于 0 ，此时应该直接返回 false：

```go
func hasValidPath(grid [][]byte) bool {
    n, m := len(grid), len(grid[0])
    if (n + m) % 2 == 0 || grid[0][0] == ')' || grid[n - 1][m - 1] == '(' {
        return false
    }
    f := make([][][]bool, n)
    for i := 0; i < n ; i ++ {
        f[i] = make([][]bool, m)
        for j := 0; j < m; j ++ {
            f[i][j] = make([]bool, (m + n + 1) / 2)
        }
    }
    var dfs func(x, y, c int) bool
    dfs = func(x, y, c int) bool {
        if c > n + m - x - y - 1 {
            return false
        }
        if x == n - 1 && y == m - 1 {
            return c == 1
        }
        if f[x][y][c] {
            return false
        }
        f[x][y][c] = true
        if grid[x][y] == '(' {
            c ++
        } else if c --; c < 0 {
            return false
        }
        return x < n - 1 && dfs(x + 1, y, c) || y < m - 1 && dfs(x, y + 1, c)
    }
    return dfs(0, 0, 0)
}
```

