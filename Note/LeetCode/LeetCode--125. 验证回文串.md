[125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/)

> 如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 **回文串** 。
>
> 字母和数字都属于字母数字字符。
>
> 给你一个字符串 `s`，如果它是 **回文串** ，返回 `true` ；否则，返回 `false` 。

----

跟验证IP差不多思路，找字符，转换，即可，不如最长回文子串

```go
func isPalindrome(s string) bool {
    var bts []byte
    for i := 0; i < len(s); i ++ {
        if s[i] >= 'A' && s[i] <= 'Z'{
            bts = append(bts, s[i] + 32)
        } else if (s[i] >= 'a' && s[i] <= 'z') || (s[i] >= '0' && s[i] <= '9') {
            bts = append(bts, s[i])
        }
    }
    for i, j := 0, len(bts) - 1; i < j; i, j = i + 1, j - 1 {
        if bts[i] != bts[j] {
            return false
        }
    }
    return true
}
```

