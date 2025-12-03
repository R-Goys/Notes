[402. 移掉 K 位数字](https://leetcode.cn/problems/remove-k-digits/)

> 给你一个以字符串表示的非负整数 `num` 和一个整数 `k` ，移除这个数中的 `k` 位数字，使得剩下的数字最小。请你以字符串形式返回这个最小的数字。

---

单调栈，最开始根本没想过这道题还可以用单调栈，太刁了，从高位开始遍历，将大于前一位的加入栈，小于前一位的，则将之前的数字弹出，也就是删除，然后再加入当前数字，这样就可以保证数字尽量地小，当然，如果遍历结束，还没有删除到k个数字，那么就可以直接去除最后几位数字，因为此时数字单调，而他们是最大的。

```go
func removeKdigits(num string, k int) string {
    stk := []byte{}
    i := 0
    for j := 0; j < len(num); j ++ {
        for i < k && len(stk) > 0 && num[j] < stk[len(stk) - 1] {
            stk = stk[:len(stk) - 1]
            i ++
        }
        stk = append(stk, num[j])
    }
    stk = stk[:len(stk)- k + i]
    for len(stk) > 0 && stk[0] == '0' {
        stk = stk[1:]
    }
    if len(stk) == 0 {
        return "0"
    }
    return string(stk)
}
```

