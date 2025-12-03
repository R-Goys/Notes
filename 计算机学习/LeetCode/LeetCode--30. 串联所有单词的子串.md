[30. 串联所有单词的子串](https://leetcode.cn/problems/substring-with-concatenation-of-all-words/)

> 给定一个字符串 `s` 和一个字符串数组 `words`**。** `words` 中所有字符串 **长度相同**。
>
>  `s` 中的 **串联子串** 是指一个包含 `words` 中所有字符串以任意顺序排列连接起来的子串。
>
> - 例如，如果 `words = ["ab","cd","ef"]`， 那么 `"abcdef"`， `"abefcd"`，`"cdabef"`， `"cdefab"`，`"efabcd"`， 和 `"efcdab"` 都是串联子串。 `"acdbef"` 不是串联子串，因为他不是任何 `words` 排列的连接。
>
> 返回所有串联子串在 `s` 中的开始索引。你可以以 **任意顺序** 返回答案。

---

最后只能通过 [:] 的方式去截取子字符串来比较，把子字符串当作元素就行了，因为我们最开始就知道所有子字符串的长度都是一样的，通过 overload 变量判断是否有超出数量的元素。

```go
func findSubstring(s string, words []string) []int {
    ans := make([]int, 0)
    strMp := make(map[string]int)
    wordLen := len(words[0])
    windowLen := wordLen * len(words)
    for _, x := range words {
        strMp[x] ++
    }
    for start := range wordLen {
        cnt := make(map[string]int)
        overload := 0
        for right := start + wordLen; right <= len(s); right += wordLen {
            inWord := s[right - wordLen : right]
            if cnt[inWord] == strMp[inWord] {
                overload ++
            }
            cnt[inWord] ++

            left := right - windowLen
            if left < 0 {
                continue
            }

            if overload == 0 {
                ans = append(ans, left)
            }

            outWord := s[left : left + wordLen]
            cnt[outWord] --
            if cnt[outWord] == strMp[outWord] {
                overload --
            }
        }
    }
    return ans
}
```

