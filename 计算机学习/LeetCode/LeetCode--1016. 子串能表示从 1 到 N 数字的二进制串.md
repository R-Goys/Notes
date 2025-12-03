[1016. 子串能表示从 1 到 N 数字的二进制串](https://leetcode.cn/problems/binary-string-with-substrings-representing-1-to-n/)

> 给定一个二进制字符串 `s` 和一个正整数 `n`，如果对于 `[1, n]` 范围内的每个整数，*其二进制表示都是 `s` 的 **子字符串** ，就返回 `true`，否则返回 `false`* 。
>
> **子字符串** 是字符串中连续的字符序列。

---

无敌了，暴力都能过：

```go
func queryString(s string, n int) bool {
    for i := n; i >= 1; i -- {
        if !strings.Contains(s, strconv.FormatUint(uint64(i), 2)) {
            return false
        }
    }
    return true
}
```

换个思路，我们从字符串中枚举子串，如果处于 [1, n]，则放入哈希表中，最终比较哈希表的长度与n的大小，此时应当是相同的

```go
func queryString(s string, n int) bool {
    mp := make(map[int]struct{})
    for i, b := range s {
        x := int(b - '0')
        if x == 0 {
            continue
        }
        for j := i + 1; x <= n; j ++ {
            mp[x] = struct{}{}
            if j == len(s) {
                break
            }
            // 此处是移位
            x = x << 1 | int(s[j] - '0')
        }
    }
    return len(mp) == n
}
```

