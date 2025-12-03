[233. 数字 1 的个数](https://leetcode.cn/problems/number-of-digit-one/)

> 给定一个整数 `n`，计算所有小于等于 `n` 的非负整数中数字 `1` 出现的个数。

---

枚举每一位，当前位置数字为1的情况，实行滚动枚举：

```go
func countDigitOne(n int) int {
    digit := 1
    high, cur, low, res := n / 10, n % 10, 0, 0
    for high != 0 || cur != 0 {
        if cur == 0 {
            res += high * digit
        } else if cur == 1 {
            // +low代表加上枚举低位的时候，cur会出现的1的个数。
            res += high * digit + low + 1
        } else {
            // cur > 1 时，cur为1的情况，需要枚举完低位，所以更换计算方式。
            res += (high + 1) * digit
        }
        low += cur * digit
        cur = high % 10
        high /= 10
        digit *= 10
    }
    return res
}
```

