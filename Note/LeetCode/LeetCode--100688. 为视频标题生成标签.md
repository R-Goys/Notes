[100688. 为视频标题生成标签](https://leetcode.cn/problems/generate-tag-for-video-caption/)

> 给你一个字符串 `caption`，表示一个视频的标题。
>
> 需要按照以下步骤 **按顺序** 生成一个视频的 **有效标签** ：
>
> 1. 将 **所有单词** 组合为单个 **驼峰命名字符串** ，并在前面加上 `'#'`。**驼峰命名字符串** 指的是除第一个单词外，其余单词的首字母大写，且每个单词的首字母之后的字符必须是小写。
> 2. **移除** 所有不是英文字母的字符，但 **保留** 第一个字符 `'#'`。
> 3. 将结果 **截断** 为最多 100 个字符。
>
> 对 `caption` 执行上述操作后，返回生成的 **标签** 。

---

针对于一切字符都需要进行大小写的判断才可以通过，有很坑的样例，写的有点屎山

```go
func generateTag(caption string) string {
    strs := make([][]byte, 0)
    pre, cur := 0, 0
    ans := "#"
    for cur < len(caption) {
        if caption[cur] == ' ' {
            if pre != cur {
                strs = append(strs, []byte(caption[pre : cur]))
            }
            pre = cur + 1
        }
        cur ++
    }
    if pre != cur {
        strs = append(strs, []byte(caption[pre : cur]))
    }

    for i, m := range strs {
        for j, x := range m {
            if i == 0 && unicode.IsUpper(rune(x)) {
                strs[i][j] += 32
            } else if i > 0 {
                if j == 0 && !unicode.IsUpper(rune(x)) {
                    strs[i][j] -= 32
                } else if j > 0 && unicode.IsUpper(rune(x)) {
                    strs[i][j] += 32
                } 
            }
        }
        ans += string(strs[i])
    }
    if len(ans) > 100 {
        return ans[:100]
    }
    return ans
}
```

