[227. 基本计算器 II](https://leetcode.cn/problems/basic-calculator-ii/)

> 给你一个字符串表达式 `s` ，请你实现一个基本计算器来计算并返回它的值。
>
> 整数除法仅保留整数部分。
>
> 你可以假设给定的表达式总是有效的。所有中间结果将在 `[-231, 231 - 1]` 的范围内。
>
> **注意：**不允许使用任何将字符串作为数学表达式计算的内置函数，比如 `eval()` 。

---

表达式求值都知道咋做，居然被这道题难住了，还是得多练练了

```go
func calculate(s string) int {
    var ans int
    var nums []int
    PreSign := '+'
    num := 0
    for i, c := range s {
        isDigit := '0' <= c && c <= '9'
        if isDigit {
            num = num * 10 + int(c - '0')
        }
        if !isDigit && s[i] != ' ' || i == len(s) - 1 {
            switch PreSign {
            case '+':
                nums = append(nums, num)
            case '-':
                nums = append(nums, -num)
            case '*':
                nums[len(nums)-1] *= num
            default:
                nums[len(nums)-1] /= num
            }
            PreSign = c
            num = 0
        }
    }
    for _, v := range nums {
        ans += v
    }
    return ans
}
```

