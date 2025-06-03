[1297. 子串的最大出现次数](https://leetcode.cn/problems/maximum-number-of-occurrences-of-a-substring/)

> 给你一个字符串 `s` ，请你返回满足以下条件且出现次数最大的 **任意** 子串的出现次数：
>
> - 子串中不同字母的数目必须小于等于 `maxLetters` 。
> - 子串的长度必须大于等于 `minSize` 且小于等于 `maxSize` 。

---

maxSize 没啥用，因为 如果 maxSize 是子串，那么 minSize 一定也是子串，所以只需要算 minSize 的定长滑动窗口即可。

```go
func maxFreq(s string, maxLetters int, minSize int, maxSize int) int {
    strMp := make(map[string]int)
    letterMp := make(map[byte]int)
    ans := 0
    for i, _ := range s {
        letterMp[s[i]] ++
        if i < minSize - 1 {
            continue
        }
        
        if len(letterMp) <= maxLetters {
            strMp[s[i - minSize + 1 : i + 1]] ++
            ans = max(ans, strMp[s[i - minSize + 1 : i + 1]])
        }
        letterMp[s[i - minSize + 1]] --
		if letterMp[s[i - minSize + 1]] <= 0 {
			delete(letterMp, s[i - minSize + 1])
		}
    }
    return ans
}
```

