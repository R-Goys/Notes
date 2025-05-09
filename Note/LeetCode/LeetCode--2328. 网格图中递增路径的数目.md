[2328. 网格图中递增路径的数目](https://leetcode.cn/problems/number-of-increasing-paths-in-a-grid/)

> 给你一个 `m x n` 的整数网格图 `grid` ，你可以从一个格子移动到 `4` 个方向相邻的任意一个格子。
>
> 请你返回在网格图中从 **任意** 格子出发，达到 **任意** 格子，且路径中的数字是 **严格递增** 的路径数目。由于答案可能会很大，请将结果对 `109 + 7` **取余** 后返回。
>
> 如果两条路径中访问过的格子不是完全相同的，那么它们视为两条不同的路径。

---

关于网格 dp ，还是 dfs 好用啊。

首先状态转移的数组全部置为 -1 ，在 dfs 中作为缓存，如果发现已经遍历过直接返回即可。

```go
var dirs = [][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}

func countPaths(grid [][]int) int {
	mod := 1000000007
	n, m:= len(grid), len(grid[0])
    ans := 0
    f := make([][]int, n)
	for i := range f {
		f[i] = make([]int, m)
		for j := range f[i] {
			f[i][j] = -1
		}
	}

	var dfs func(int, int) int
	dfs = func(i, j int) int {
		if f[i][j] != -1 {
			return f[i][j]
		}
		res := 1
		for _, dir := range dirs {
            x, y := i + dir[0], j + dir[1];
			if 0 <= x && x < n && 0 <= y && y < m && grid[x][y] > grid[i][j] {
				res = (res + dfs(x, y)) % mod
			}
		}
		f[i][j] = res
		return res
	}

    for i := 0; i < n; i ++ {
        for j := 0; j < m; j ++ {
            ans = (ans + dfs(i, j)) % mod
        }
    }
    return ans
}
```

