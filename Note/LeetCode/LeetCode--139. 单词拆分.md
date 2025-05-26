[139. 单词拆分](https://leetcode.cn/problems/word-break/)

> 给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。
>
> **注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

---

一眼dp，直接写就行了

```go
func wordBreak(s string, wordDict []string) bool {
    n := len(s)
    m := len(wordDict)
    f := make([]bool, n + 1)
    f[0] = true
    for i := 1; i <= n; i ++ {
        for j := 0; j < m; j ++ {
            if i >= len(wordDict[j]) && s[i - len(wordDict[j]) : i] == wordDict[j] {
                f[i] = f[i - len(wordDict[j])] || f[i]
            }
        }
    }
    return f[n]
}
```

二刷，秒

```go
func wordBreak(s string, wordDict []string) bool {
    n, m := len(s), len(wordDict)
    f := make([]bool, n + 1)
    f[0] = true
    for i := 1; i <= n; i ++ {
        for j := 0; j < m; j ++ {
            if i >= len(wordDict[j]) && s[i - len(wordDict[j]) : i] == wordDict[j] {
                f[i] = f[i] || f[i - len(wordDict[j])]
            }
        }
    }
    return f[n]
}
```

