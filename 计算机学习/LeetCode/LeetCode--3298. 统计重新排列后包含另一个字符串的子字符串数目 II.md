[3298. 统计重新排列后包含另一个字符串的子字符串数目 II](https://leetcode.cn/problems/count-substrings-that-can-be-rearranged-to-contain-a-string-ii/)

> 给你两个字符串 `word1` 和 `word2` 。
>
> 如果一个字符串 `x` 重新排列后，`word2` 是重排字符串的 前缀 ，那么我们称字符串 `x` 是 **合法的** 。
>
> 请你返回 `word1` 中 **合法** 子字符串 的数目。
>
> **注意** ，这个问题中的内存限制比其他题目要 **小** ，所以你 **必须** 实现一个线性复杂度的解法。

---

先统计 word2 中的字符总数，存为哈希表，然后滑窗在 word1 里面找这些字符串

```go
func validSubstringCount(word1 string, word2 string) int64 {
    target := ['z' + 1]int{}
    cnt := 0
    for _, x := range word2 {
        if target[x] == 0 {
            cnt ++
        }
        target[x] ++
    }
    l := 0
    ans := 0
    for r, x := range word1 {
        target[x] --
        if target[x] == 0 {
            cnt --
        }
        for cnt == 0 {
            ans += len(word1) - r
            if target[word1[l]] == 0 {
                cnt ++
            }
            target[word1[l]] ++
            l ++
        }
    }
    return int64(ans)
}
```

