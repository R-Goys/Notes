[3561. 移除相邻字符](https://leetcode.cn/problems/resulting-string-after-adjacent-removals/)

> 给你一个由小写英文字母组成的字符串 `s`。
>
> 你 **必须** 在字符串 `s` 中至少存在两个 **连续** 字符时，反复执行以下操作：
>
> - 移除字符串中 **最左边** 的一对按照字母表 **连续** 的相邻字符（无论是按顺序还是逆序，例如 `'a'` 和 `'b'`，或 `'b'` 和 `'a'`）。
> - 将剩余字符向左移动以填补空隙。
>
> 当无法再执行任何操作时，返回最终的字符串。
>
> **注意：**字母表是循环的，因此 `'a'` 和 `'z'` 也视为连续。

---

鱼鱼了，刷了 300 道题，算术评级 3 的都写不出来，看来是 dp 刷傻了，周赛的时候还以为用 dp 很简单，这里直接用栈来判断，邻项消除，是一个经典的栈的应用。

```go
func resultingString(s string) string {
    st := []byte{}
    for _, b := range s {
        st = append(st, byte(b))
        if len(st) > 1 && Check(st[len(st) - 1], st[len(st) - 2]) {
            st = st[:len(st) - 2]
        }
    }
    return string(st)
}

func Check(a, b byte) bool {
	d := abs(int(a) - int(b))
	return d == 1 || d == 25
}

func abs(x int) int {
    if x > 0 {
        return x
    }
    return - x
}
```

