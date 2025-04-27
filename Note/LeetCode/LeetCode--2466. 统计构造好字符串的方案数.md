[2466. 统计构造好字符串的方案数](https://leetcode.cn/problems/count-ways-to-build-good-strings/)

> 给你整数 `zero` ，`one` ，`low` 和 `high` ，我们从空字符串开始构造一个字符串，每一步执行下面操作中的一种：
>
> - 将 `'0'` 在字符串末尾添加 `zero` 次。
> - 将 `'1'` 在字符串末尾添加 `one` 次。
>
> 以上操作可以执行任意次。
>
> 如果通过以上过程得到一个 **长度** 在 `low` 和 `high` 之间（包含上下边界）的字符串，那么这个字符串我们称为 **好** 字符串。
>
> 请你返回满足以上要求的 **不同** 好字符串数目。由于答案可能很大，请将结果对 `109 + 7` **取余** 后返回。

---

也是一道背包变种题型，直接秒，开辟两个一维空间，表示以某一种操作结尾，长度为 i 的操作数，直接dp即可。

记得取模之后的减法也可能会出现负数，所以需要加个mod。

```go
func countGoodStrings(low int, high int, zero int, one int) int {
    f := make([][2]int, high + 1)
    mod := 1000000007
    for i := 1; i <= high; i ++ {
        if i - zero >= 0 {
            f[i][0] = (f[i - zero][1] + f[i - zero][0]) % mod + 1
        }
        if i - one >= 0 {
            f[i][1] = (f[i - one][1] + f[i - one][0]) % mod + 1
        }
    }
    return ((f[high][1] + f[high][0]) % mod  - (f[low - 1][0] + f[low - 1][1]) % mod + mod) % mod
}
```

