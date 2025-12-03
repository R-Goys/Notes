[2510. 检查是否有路径经过相同数量的 0 和 1](https://leetcode.cn/problems/check-if-there-is-a-path-with-equal-number-of-0s-and-1s/)

> 给定一个 **下标从 0 开始** 的 `m x n` 的 **二进制** 矩阵 `grid` ，从坐标为 `(row, col)` 的元素可以向右走 `(row, col+1)` 或向下走 `(row+1, col)` 。
>
> 返回一个布尔值，表示从 `(0, 0)` 出发是否存在一条路径，经过 **相同** 数量的 `0` 和 `1`，到达终点 `(m-1, n-1)` 。如果存在这样的路径返回 `true` ，否则返回 `false` 。

---

看 py 的代码可以通过位运算解决，如果使用 golang 里面的 bigint 也可以通过这道题，姑且写一下关于位运算的解法

```go
func isThereAPath(grid [][]int) bool {
	m, n := len(grid), len(grid[0])
	c := m + n - 1
	if c&1 != 0 {
		return false
	}
	c >>= 1

	dp := make([]*big.Int, n)
	for i := 0; i < n; i++ {
		dp[i] = new(big.Int)
	}
	dp[0].SetInt64(1)

	for i := 0; i < m; i ++ {
		for j := 0; j < n; j ++ {
			if j > 0 {
				dp[j].Or(dp[j], dp[j-1])
			}
			if grid[i][j] == 1 {
				dp[j].Lsh(dp[j], 1) 
			}
		}
	}

	mask := new(big.Int).Lsh(big.NewInt(1), uint(c))
	return dp[n - 1].And(dp[n - 1], mask).Cmp(big.NewInt(0)) != 0
}

```

但是这个方法涉及到很多平常用不到的库，了解一下即可，最好使用另一种

这种的思路就是我们统计走到当前路径时候，最少的 1 的数量以及最多的 1 的数量，如果这两个变量构成的值域包含了路径经过所有元素的一半，那么则说明我们存在相同数量的 0 和 1 ：

```go
func isThereAPath(grid [][]int) bool {
	n, m := len(grid), len(grid[0])
	t := (n + m - 1) / 2
	if (n + m - 1) % 2 != 0 {
		return false
	}
    mi, ma := make([][]int, n), make([][]int, n)

	for i := 0; i < n; i ++ {
        mi[i] = make([]int, m)
        ma[i] = make([]int, m)
		for j := 0; j < m; j ++ {
			mi[i][j] = 0x3f3f3f3f
		}
	}

	mi[0][0] = grid[0][0]
	ma[0][0] = grid[0][0]

	for i := 0; i < n; i ++ {
		for j := 0; j < m; j ++ {
			x := grid[i][j]
			if i >= 1 {
				upd_mi(&mi[i][j], mi[i - 1][j] + x)
				upd_ma(&ma[i][j], ma[i - 1][j] + x)
			}
			if j >= 1 {
				upd_mi(&mi[i][j], mi[i][j - 1] + x)
				upd_ma(&ma[i][j], ma[i][j - 1] + x)
			}
		}
	}

	return mi[n - 1][m - 1] <= t && ma[n - 1][m - 1] >= t
}

func upd_mi(x *int, y int) {
	if y < *x {
		*x = y
	}
}

func upd_ma(x *int, y int) {
	if y > *x {
		*x = y
	}
}
```

