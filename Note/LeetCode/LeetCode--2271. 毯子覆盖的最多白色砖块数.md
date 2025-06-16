[2271. 毯子覆盖的最多白色砖块数](https://leetcode.cn/problems/maximum-white-tiles-covered-by-a-carpet/)

> 给你一个二维整数数组 `tiles` ，其中 `tiles[i] = [li, ri]` ，表示所有在 `li <= j <= ri` 之间的每个瓷砖位置 `j` 都被涂成了白色。
>
> 同时给你一个整数 `carpetLen` ，表示可以放在 **任何位置** 的一块毯子的长度。
>
> 请你返回使用这块毯子，**最多** 可以盖住多少块瓷砖。

---

依旧滑窗，不过是以块为单位，一块一块滑动，而不是一格一格，因为对于每一块为起点的区间，必定是覆盖当前块时可以覆盖的最大区间面积之一，证明略，就是贪心思想。

```go
func maximumWhiteTiles(tiles [][]int, carpetLen int) int {
        slices.SortFunc(tiles, func(a, b []int) int {
            return a[0] - b[0]
        })
        cover, left, ans := 0, 0, 0
        for _, tile := range tiles {
            tl, tr := tile[0], tile[1]
            cover += tr - tl + 1
            for tiles[left][1] + carpetLen - 1 < tr {
                cover -= tiles[left][1] - tiles[left][0] + 1
                left ++
            }
            uncover := max(tr - carpetLen + 1 - tiles[left][0], 0)
            ans = max(ans, cover - uncover)
        }
    return ans
}
```

