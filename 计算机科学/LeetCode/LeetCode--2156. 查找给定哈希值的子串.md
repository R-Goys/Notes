[2156. 查找给定哈希值的子串](https://leetcode.cn/problems/find-substring-with-given-hash-value/)

> 给定整数 `p` 和 `m` ，一个长度为 `k` 且下标从 **0** 开始的字符串 `s` 的哈希值按照如下函数计算：
>
> - `hash(s, p, m) = (val(s[0]) * p0 + val(s[1]) * p1 + ... + val(s[k-1]) * pk-1) mod m`.
>
> 其中 `val(s[i])` 表示 `s[i]` 在字母表中的下标，从 `val('a') = 1` 到 `val('z') = 26` 。
>
> 给你一个字符串 `s` 和整数 `power`，`modulo`，`k` 和 `hashValue` 。请你返回 `s` 中 **第一个** 长度为 `k` 的 **子串** `sub` ，满足 `hash(sub, power, modulo) == hashValue` 。
>
> 测试数据保证一定 **存在** 至少一个这样的子串。
>
> **子串** 定义为一个字符串中连续非空字符组成的序列。

---

通过正序遍历计算，我这里过不了样例，看了 0 神的才知道要倒序计算哈希值，因为计算方式已经给出来了，所以还是比较简单的。

```go
func subStrHash(s string, power, mod, k, hashValue int) (ans string) {
    n := len(s)
    hash, pk := 0, 1
    for i := n - 1; i >= n - k; i -- {
        hash = (hash * power + int(s[i] - 'a' + 1)) % mod
        pk = pk * power % mod
    }
    if hash == hashValue {
        ans = s[n - k :]
    }
    for i := n - k - 1; i >= 0; i -- {
        hash = (hash * power + int(s[i] - 'a' + 1) - pk * int(s[i + k] - 'a' + 1) % mod + mod) % mod
        if hash == hashValue {
            ans = s[i : i + k]
        }
    }
    return
}
```

