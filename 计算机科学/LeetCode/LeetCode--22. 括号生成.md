[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

> 数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

----

### 简单dfs，无需多说

```go
func generateParenthesis(n int) []string {
    var ans []string
    dfs(n, 0, 0, "", &ans)
    return ans
}

func dfs(n int, left int, right int, cur string, ans *[]string) {
    if left == n && right == n {
        *ans = append(*ans, cur)
        return
    }

    if left == n {
        dfs(n, left, right + 1, cur + ")", ans)
        return
    }
    if left == right {
        dfs(n, left + 1, right, cur + "(", ans)
        return
    }

    dfs(n, left + 1, right, cur + "(", ans)
    dfs(n, left, right + 1, cur + ")", ans)
    return
}

```

