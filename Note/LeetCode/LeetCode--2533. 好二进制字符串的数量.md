[2533. 好二进制字符串的数量](https://leetcode.cn/problems/number-of-good-binary-strings/)

> 给你四个整数 `minLength`、`maxLength`、`oneGroup` 和 `zeroGroup` 。
>
> **好** 二进制字符串满足下述条件：
>
> - 字符串的长度在 `[minLength, maxLength]` 之间。
> - 每块连续 1 的个数是 oneGroup 的整数倍
>   - 例如在二进制字符串 `00***11***0***1111***00` 中，每块连续 `1` 的个数分别是`[2,4]` 。
> - 每块连续 0 的个数是 zeroGroup 的整数倍
>   - 例如在二进制字符串 `***00***11***0***1111***00***` 中，每块连续 `0` 的个数分别是 `[2,1,2]` 。
>
> 请返回 **好** 二进制字符串的个数。答案可能很大**，**请返回对 `109 + 7` **取余** 后的结果。
>
> **注意：**`0` 可以被认为是所有数字的倍数。

---

之前的求好字符串原题

```go
func goodBinaryStrings(minLength int, maxLength int, oneGroup int, zeroGroup int) int {
    f := make([][2]int, maxLength + 1)
    mod := 1000000007
    for i := 0; i <= maxLength; i ++ {
        if i >= oneGroup {
            f[i][1] = (f[i - oneGroup][1] + f[i - oneGroup][0] + 1) % mod
        }
        if i >= zeroGroup {
            f[i][0] = (f[i - zeroGroup][1] + f[i - zeroGroup][0] + 1) % mod
        }
    }
    return ((f[maxLength][0] + f[maxLength][1]) % mod - (f[minLength - 1][1] + f[minLength - 1][0]) % mod + mod) % mod
}
```

