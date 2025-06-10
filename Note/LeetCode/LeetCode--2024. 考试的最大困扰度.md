[2024. 考试的最大困扰度](https://leetcode.cn/problems/maximize-the-confusion-of-an-exam/)

> 一位老师正在出一场由 `n` 道判断题构成的考试，每道题的答案为 true （用 `'T'` 表示）或者 false （用 `'F'` 表示）。老师想增加学生对自己做出答案的不确定性，方法是 **最大化** 有 **连续相同** 结果的题数。（也就是连续出现 true 或者连续出现 false）。
>
> 给你一个字符串 `answerKey` ，其中 `answerKey[i]` 是第 `i` 个问题的正确结果。除此以外，还给你一个整数 `k` ，表示你能进行以下操作的最多次数：
>
> - 每次操作中，将问题的正确答案改为 `'T'` 或者 `'F'` （也就是将 `answerKey[i]` 改为 `'T'` 或者 `'F'` ）。
>
> 请你返回在不超过 `k` 次操作的情况下，**最大** 连续 `'T'` 或者 `'F'` 的数目。

---

和周赛遇到的一道题很像，好像是第一道，也是给定了两个字符关于数组的问题，这里就借用灵神的思想了。

```go
func maxConsecutiveAnswers(answerKey string, k int) int {
    var solve func(c rune) int
    solve = func(c rune) int {
        mp := make(map[rune]int)
        l := 0
        ans := 0
        for r, x := range answerKey {
            mp[x] ++
            for mp[c] > k {
                mp[rune(answerKey[l])] --
                l ++
            }
            ans = max(ans, r - l + 1)
        }
        return ans
    }
    return max(solve('T'), solve('F'))
}
```

