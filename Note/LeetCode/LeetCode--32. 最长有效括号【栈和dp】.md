[32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)

----

## 前言

分享一下dp和栈两个方法

## 正文

>给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号子串的长度。

这道题与[20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)类似，但是这道题需要计算出最长的有效括号字串的长度，所以做法并不完全一样。

### 动态规划

该题目dp方法最难的就是得出状态转移方程，只要克服了这一点，剩下都很简单，下面，我们以字符串`"((())()("`为例子。

从左向右遍历，设定f[i]为**包含当前下标对应的位置**的最长有效括号的长度，所以f[n]并不是我们最终的答案，因此我们需要一个变量**ans**来存储最大的值，既然如此，那么f[i]如何表示呢？
形式如`"?()"`，当遍历到')'时，我们可以令'?'对应的下标对应的值 + 2，就是我们当前的f[i]，形式如`"?((...))"`，则在遍历到最后一个括号时，则`f[last] = f[last - 1] + f[?] + 2`即可，于是我们的代码就可以写出来了：

```go
func longestValidParentheses(s string) int {
    n := len(s)
    f := make([]int, n + 1)
    ans := 0
    for i := 1; i < n; i++ {
        if s[i] == ')' {
            if s[i - 1] == '(' {
                f[i + 1] = f[i - 1] + 2
            } else {
                j := i - f[i] - 1
                if j >= 0 && s[j] == '(' {
                    f[i + 1] = f[j] + f[i] + 2
                }
            }
            ans = max(ans, f[i + 1])
        }
    }
    return ans
}
```

### 栈

栈的难点在于思考在出栈，入栈时应该如何计算ans的值，事实上，只需要在遍历到'('时，将下标存入栈中即可。

同时为了在出栈之后能够正确的计算我们的有效括号长度，我们还需要提前加入一个哨兵元素，来便于计算，同时，出栈之后，当栈为空时，我们还需要继续入栈一个哨兵(当前遍历元素的下标)，而计算括号长度则：`i - st.top() - 1`

然后就可以写出代码：

```go
func longestValidParentheses(s string) int {
    n := len(s)
    var st []int
    ans := 0
    st = append(st, -1)
    for i := 0; i < n; i++ {
        if s[i] == '(' {
            st = append(st, i)
            continue
        }
        st = st[:len(st) - 1]
        if len(st) == 0 {
            st = append(st, i)
        } else {
            ans = max(ans, i - st[len(st) - 1])
        }
    }
    return ans
}
```

----

