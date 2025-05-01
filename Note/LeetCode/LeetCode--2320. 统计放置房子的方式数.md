[2320. 统计放置房子的方式数](https://leetcode.cn/problems/count-number-of-ways-to-place-houses/)

> 一条街道上共有 `n * 2` 个 **地块** ，街道的两侧各有 `n` 个地块。每一边的地块都按从 `1` 到 `n` 编号。每个地块上都可以放置一所房子。
>
> 现要求街道同一侧不能存在两所房子相邻的情况，请你计算并返回放置房屋的方式数目。由于答案可能很大，需要对 `109 + 7` 取余后再返回。
>
> 注意，如果一所房子放置在这条街某一侧上的第 `i` 个地块，不影响在另一侧的第 `i` 个地块放置房子。

---

原本只是抱着试试的心态去把一边街道的种类数结果乘起来，没想到真的过了。。

```go
func countHousePlacements(n int) int {
    f := make([][2]int, n + 1)
    f[0][0], f[0][1] = 1, 0
    mod := 1000000007
    for i := 1; i <= n ; i ++ {
        f[i][0], f[i][1] = (f[i - 1][1] + f[i - 1][0]) % mod, f[i - 1][0]
    }    
    return ((f[n][1] + f[n][0]) % mod) * ((f[n][1] + f[n][0]) % mod) % mod
}
```

另一种 0 神的最终计算都是一种方式，但是他使用了线性 dp 去计算一边街道的方式数：

```go
const mod = 1000000007

var f = [10001]int{1, 2}

func init() {
	for i := 2; i < len(f); i++ {
		f[i] = (f[i - 1] + f[i - 2]) % mod
	}
}

func countHousePlacements(n int) int {
	return f[n] * f[n] % mod
}
```

