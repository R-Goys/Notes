[3602. 十六进制和三十六进制转化](https://leetcode.cn/problems/hexadecimal-and-hexatrigesimal-conversion/)

> 给你一个整数 `n`。
>
> 返回 `n2` 的 **十六进制表示** 和 `n3` 的 **三十六进制表示** 拼接成的字符串。
>
> **十六进制** 数定义为使用数字 `0 – 9` 和大写字母 `A - F` 表示 0 到 15 的值。
>
> **三十六进制** 数定义为使用数字 `0 – 9` 和大写字母 `A - Z` 表示 0 到 35 的值。

---

进制转换，纯语法题，只需要注意细节就行了。

```go
func concatHex36(n int) string {
    n16 := ""
    cur := n * n
    for cur > 0 {
        mod := cur % 16
        if mod >= 10 {
            n16 = string('A' + mod - 10) + n16
        } else {
            n16 = string('0' + mod) + n16
        }
        cur /= 16
    }
    n36 := ""
    cur = n * n * n
    for cur > 0 {
        mod := cur % 36
        if mod >= 10 {
            n36 = string('A' + mod - 10) + n36
        } else {
            n36 = string('0' + mod) + n36
        }
        cur /= 36
    }
    return n16 + n36
}
```

