[151. 反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

> 给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。
>
> **单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。
>
> 返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。
>
> **注意：**输入字符串 `s`中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

----

首先是一个暴力做法，每一轮循环跳过前导零，然后找到字符串，用i，j两个变量维护一个单词。

神经，我看题解直接用内置函数给他分解了，闹麻了。

```go
func reverseWords(s string) string {
    var temp []string
    i, j := 0, 0
    for j < len(s) {
        for j < len(s) && s[j] == ' ' {
            j ++
        }
        i = j
        if j < len(s) && s[j] != ' '{
            for j < len(s) && s[j] != ' ' {
                j ++
            }
            temp = append(temp, s[i:j])
            i = j
        }
    }
    var ans string
    for i := len(temp) - 1; i >= 0; i -- {
        ans += temp[i]
        if i != 0 {
            ans += " "
        }
    }
    return ans
}
```

另一种比较简单易懂的方法，无需栈保存单词

```go
func reverseWords(s string) (res string) {
    s = " " + s + " "
    l, r := len(s) - 1, len(s) - 1
    for i := len(s) - 2; i >= 0; i--{
        if s[i] == ' '{
            l, r = i, l
            if r > l + 1{
                res = res + s[l + 1:r] + " "
            }
        }
    }
    return res[:len(res) - 1]
}
```

