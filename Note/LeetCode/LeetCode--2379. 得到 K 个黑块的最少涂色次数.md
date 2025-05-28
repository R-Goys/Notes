[2379. 得到 K 个黑块的最少涂色次数](https://leetcode.cn/problems/minimum-recolors-to-get-k-consecutive-black-blocks/)

> 给你一个长度为 `n` 下标从 **0** 开始的字符串 `blocks` ，`blocks[i]` 要么是 `'W'` 要么是 `'B'` ，表示第 `i` 块的颜色。字符 `'W'` 和 `'B'` 分别表示白色和黑色。
>
> 给你一个整数 `k` ，表示想要 **连续** 黑色块的数目。
>
> 每一次操作中，你可以选择一个白色块将它 **涂成** 黑色块。
>
> 请你返回至少出现 **一次** 连续 `k` 个黑色块的 **最少** 操作次数。

---

双指针和遍历都能做，我这里直接用遍历，思想都是一样的：

```go
func minimumRecolors(blocks string, k int) int {
    cnt, ans := 0, 114514
    for i, x := range blocks {
        if x == 'W' {
            cnt ++
        }
        if i + 1 < k {
            continue
        }
        ans = min(cnt, ans)
        if blocks[i - k + 1] == 'W' {
            cnt --
        }
    }
    return ans
}
```

