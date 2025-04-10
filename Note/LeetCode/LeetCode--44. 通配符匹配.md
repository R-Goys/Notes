[44. 通配符匹配](https://leetcode.cn/problems/wildcard-matching/)

> 给你一个输入字符串 (`s`) 和一个字符模式 (`p`) ，请你实现一个支持 `'?'` 和 `'*'` 匹配规则的通配符匹配：
>
> - `'?'` 可以匹配任何单个字符。
> - `'*'` 可以匹配任意字符序列（包括空字符序列）。
>
> 判定匹配成功的充要条件是：字符模式必须能够 **完全匹配** 输入字符串（而不是部分匹配）。

---

一连两个hard，还好这道题和之前那个正则表达式匹配很像，理解起来比较容易：

```go
func isMatch(s string, p string) bool {
    n, m := len(s), len(p)
    f := make([][]bool, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]bool, m + 1)
    }
    f[0][0] = true
    for j := 1; j <= m; j ++ {
        if p[j - 1] == '*' {
            //初始化，如果p有前导的*，全部为true
            f[0][j] = true
        } else {
            break
        }
    }   
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if p[j - 1] == '*' {
                //如果遇到*，则根据前两个状态进行更新，分别对应着没有通配符为空和通配符匹配一个字符串(前面传递的)
                f[i][j] = f[i][j - 1] || f[i - 1][j]
            } else if p[j - 1] == '?' || s[i - 1] == p[j - 1] {
                f[i][j] = f[i - 1][j - 1]
            }
        }
    }
    return f[n][m]
}
```

