[224. 基本计算器](https://leetcode.cn/problems/basic-calculator/)

> 给你一个字符串表达式 `s` ，请你实现一个基本计算器来计算并返回它的值。
>
> 注意:不允许使用任何将字符串作为数学表达式计算的内置函数，比如 `eval()` 。

---

这道题真牛逼，坑太多了，比如说负号可以作为符号位，运算式中只有加减号，让我重新写了好多次😶‍🌫️

不过这道题确实不错，我以前不知道这种负号作符号位是怎么算的，所以也算学到了新东西吧。

思路则把符号位维护成一个整型切片，遇到括号的时候，就进行弹出或者压入操作，以便可以正确更新符号位，遇到负号则根据外面的括号来进行判断是正还是负，随后加入ans，通俗易懂一点就是将括号展开，使得便于计算。

```go
func calculate(s string) int {
    ops := []int{1}
    sign := 1
    ans := 0
    n := len(s)
    i := 0
    for i < n {
        switch s[i] {
        case ' ':
            i++
        case '+':
            sign = ops[len(ops)-1]
            i++
        case '-':
            sign = -ops[len(ops)-1]
            i++
        case '(':
            ops = append(ops, sign)
            i++
        case ')':
            ops = ops[:len(ops)-1]
            i++
        default:
            num := 0
            for ; i < n && '0' <= s[i] && s[i] <= '9'; i++ {
                num = num*10 + int(s[i]-'0')
            }
            ans += sign * num
        }
    }
    return ans
}
```

