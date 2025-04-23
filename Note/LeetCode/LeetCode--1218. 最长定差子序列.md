[1218. 最长定差子序列](https://leetcode.cn/problems/longest-arithmetic-subsequence-of-given-difference/)

> 给你一个整数数组 `arr` 和一个整数 `difference`，请你找出并返回 `arr` 中最长等差子序列的长度，该子序列中相邻元素之间的差等于 `difference` 。
>
> **子序列** 是指在不改变其余元素顺序的情况下，通过删除一些元素或不删除任何元素而从 `arr` 派生出来的序列。

---

之前用了双重for循环，结果超时了，发现直接用背包问题的思路就行了，草

```go
func longestSubsequence(arr []int, difference int) (ans int) {
    f := map[int]int{}
    for _, v := range arr {
        f[v] = f[v - difference] + 1
        ans = max(f[v], ans)
    }
    return
}
```

