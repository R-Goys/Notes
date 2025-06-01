[2107. 分享 K 个糖果后独特口味的数量](https://leetcode.cn/problems/number-of-unique-flavors-after-sharing-k-candies/)

> 您将获得一个 **从0开始的** 整数数组 `candies` ，其中 `candies[i]` 表示第 `i` 个糖果的味道。你妈妈想让你和你妹妹分享这些糖果，给她 `k` 个 **连续** 的糖果，但你想保留尽可能多的糖果口味。
> 在与妹妹分享后，返回 **最多** 可保留的 **独特** 口味的糖果。

----

借助两个哈希表来记录每种糖果的数量，并且通过滑动窗口遍历计算：

```go
func shareCandies(candies []int, k int) int {
    AllMp := make(map[int]int, 0)
    PickMp := make(map[int]int, 0)
    total := 0
    pick := 0
    ans := 114514
    for _, x := range candies {
        if AllMp[x] == 0 {
            total ++
        }
        AllMp[x] ++
    }
    if k == 0 {
        return total
    }
    for i, x := range candies {
        PickMp[x] ++
        if PickMp[x] == AllMp[x] {
            pick ++
        }
        if i < k - 1 {
            continue
        }
        ans = min(pick, ans)
        if PickMp[candies[i - k + 1]] == AllMp[candies[i - k + 1]] {
            pick --
        }
        PickMp[candies[i - k + 1]] --
    }
    return total - ans
}
```

