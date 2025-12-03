[739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)

> 给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

---

栈，一次正序遍历。

```go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    st := make([]int, 0)
    ans := make([]int, len(temperatures))
    
    for i := 0; i < n; i ++ {
        if len(st) == 0 {
            st = append(st, i)
        }
        for len(st) != 0 && temperatures[st[len(st) - 1]] < temperatures[i] {
            top := st[len(st) - 1]
            ans[top] = i - top
            st = st[:len(st) - 1]
        }
        st = append(st, i)
    }
    return ans
}
```

二刷，二分写烦了缓解一下心情

```go
func dailyTemperatures(temperatures []int) []int {
    ans := make([]int, len(temperatures))
    stk := make([]int, 0)
    for i, x := range temperatures {
        for len(stk) > 0 && temperatures[stk[len(stk) - 1]] < x {
            idx := stk[len(stk) - 1]
            stk = stk[:len(stk) - 1]
            ans[idx] = i - idx
        }
        stk = append(stk, i)
    }
    return ans
}
```

