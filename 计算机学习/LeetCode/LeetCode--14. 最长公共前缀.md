[14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)

>编写一个函数来查找字符串数组中的最长公共前缀。
>
>如果不存在公共前缀，返回空字符串 `""`。

---

## 正文	

最开始还想用trie树做，但发现忘完了，于是直接暴力扫描：

```go
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    for i := 0; i < len(strs[0]); i++ {
        for j := 1; j < len(strs); j++ {
            if i == len(strs[j]) || strs[j][i] != strs[0][i] {
                return strs[0][:i]
            }
        }
    }
    return strs[0]
}
```

二刷：

写出来竟然一模一样.哈哈哈

```go
func longestCommonPrefix(strs []string) string {
    for i := 0; i < len(strs[0]); i ++ {
        for j := 0; j < len(strs); j ++ {
            if len(strs[j]) == i || strs[j][i] != strs[0][i] {
                return strs[0][:i]
            }
        }
    }
    return strs[0]
}
```

