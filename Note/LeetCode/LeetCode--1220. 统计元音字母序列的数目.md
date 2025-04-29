[1220. 统计元音字母序列的数目](https://leetcode.cn/problems/count-vowels-permutation/)

> 给你一个整数 `n`，请你帮忙统计一下我们可以按下述规则形成多少个长度为 `n` 的字符串：
>
> - 字符串中的每个字符都应当是小写元音字母（`'a'`, `'e'`, `'i'`, `'o'`, `'u'`）
> - 每个元音 `'a'` 后面都只能跟着 `'e'`
> - 每个元音 `'e'` 后面只能跟着 `'a'` 或者是 `'i'`
> - 每个元音 `'i'` 后面 **不能** 再跟着另一个 `'i'`
> - 每个元音 `'o'` 后面只能跟着 `'i'` 或者是 `'u'`
> - 每个元音 `'u'` 后面只能跟着 `'a'`
>
> 由于答案可能会很大，所以请你返回 模 `10^9 + 7` 之后的结果。

---

没意思，就是考你会不会读题

```go
const (
    A = 0
    E = 1
    I = 2
    O = 3
    U = 4
)
func countVowelPermutation(n int) int {
    f := make([][5]int, n)
    f[0][A], f[0][E], f[0][I], f[0][O], f[0][U] = 1, 1, 1, 1, 1
    mod := 1000000007
    for i := 1; i < n; i ++ {
        f[i][A] = (f[i - 1][I] + f[i - 1][U] + f[i - 1][E]) % mod
        f[i][E] = (f[i - 1][A] + f[i - 1][I]) % mod
        f[i][I] = (f[i - 1][E] + f[i - 1][O]) % mod
        f[i][O] = f[i - 1][I] % mod
        f[i][U] = (f[i - 1][O] + f[i - 1][I]) % mod
    }
    ans := 0
    for i := 0; i < 5; i ++ {
        ans = (f[n - 1][i] + ans) % mod
    }
    return ans
}
```

