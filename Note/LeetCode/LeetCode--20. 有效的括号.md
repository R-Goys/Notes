[20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

> 给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。
>
> 有效字符串需满足：
>
> 1. 左括号必须用相同类型的右括号闭合。
> 2. 左括号必须以正确的顺序闭合。
> 3. 每个右括号都有一个对应的相同类型的左括号。

之前写过一次，用的是栈，今天一看见就知道咋做了

```go
func isValid(s string) bool {
    var Stack []rune
    for _, c := range s {
        if c == '[' || c == '(' || c == '{' {
            Stack = append(Stack, c)
        }else {
            if len(Stack) == 0 {
                return false
            }
            j := Stack[len(Stack) - 1]
            if (c == ']' && j == '[') || (c == '}' && j == '{') || (c == ')' && j == '(') {
                Stack = Stack[:len(Stack) - 1]
            } else {
                return false
            }
        }
    }
    if len(Stack) > 0 {
        return false
    }
    return true
}
```

