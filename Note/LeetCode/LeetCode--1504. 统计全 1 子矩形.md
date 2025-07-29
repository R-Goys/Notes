[1504. 统计全 1 子矩形](https://leetcode.cn/problems/count-submatrices-with-all-ones/)

> 给你一个 `m x n` 的二进制矩阵 `mat` ，请你返回有多少个 **子矩形** 的元素全部都是 1 。

---

并非 mid 题，思路是一行一行遍历，统计当前格子自底向上形成了多长的柱子，如果遇到 0，则是这根柱的结束，每一行作为单独的 heights 数组，进行单调栈计算，且仅会计算包含了当前柱子的元素的矩形，这里的公式思考一下就明白了。

```go
func numSubmat(mat [][]int) int {
    heights := make([]int, len(mat[0]))
    ans := 0
    for _, row := range mat {
        for j, x := range row {
            if x == 0 {
                heights[j] = 0
            } else {
                heights[j] ++
            }
        }

        type tuple struct{ j, f, h int } 
        st := []tuple{{-1, 0, -1}}
        for j, h := range heights {
            for st[len(st) - 1].h >= h {
                st = st[:len(st) - 1]
            }
            p := st[len(st) - 1]
            left, f := p.j, p.f
            f += (j - left) * h
            ans += f
            st = append(st, tuple{j, f, h})
        } 
    }
    return ans
}
```

