[1044. 最长重复子串](https://leetcode.cn/problems/longest-duplicate-substring/)

> 给你一个字符串 `s` ，考虑其所有 *重复子串* ：即 `s` 的（连续）子串，在 `s` 中出现 2 次或更多次。这些出现之间可能存在重叠。
>
> 返回 **任意一个** 可能具有最长长度的重复子串。如果 `s` 不含重复子串，那么答案为 `""` 。

这道题代码看起来有点复杂，但是实际上是通过二分查找最长的重复子串的长度，然后将长度传入check进行滚动哈希，看看是否有对应长度的重复数组，并且此处为了减少哈希碰撞，还使用了两个种子进行哈希计算。

```c
func randInt(a, b int) int {
    return a + rand.Intn(b-a)
}

func pow(x, n , mod int) int {
    res := 1
    for ; n > 0; n >>= 1 {
        if n & 1 > 0 {
            res = res * x % mod
        }
        x = x * x % mod
    }
    return res
}
//长度为m的子数组是否存在
func check(arr []byte, m, a1, a2, mod1, mod2 int) int {
    al1, al2 := pow(a1, m , mod1), pow(a2, m ,mod2)
    h1, h2 := 0, 0
    //初始化我们最开始的起始哈希值
    for _, c := range arr[:m] {
        h1 = (h1 * a1 + int(c)) % mod1
        h2 = (h2 * a2 + int(c)) % mod2
    }
    seen := map[[2]int]bool{{h1, h2}: true}
    for start := 1; start <= len(arr) - m; start ++ {
        //减去表示滚出去，加上表示新加入的字母
        h1 = (h1 * a1 - int(arr[start - 1]) * al1 + int(arr[start + m - 1])) % mod1
        if h1 < 0 {
            h1 += mod1
        }
        h2 = (h2 * a2 - int(arr[start - 1]) * al2 + int(arr[start + m - 1])) % mod2
        if h2 < 0 {
            h2 += mod2
        }
        if seen[[2]int{h1, h2}] {
            return start
        }
        seen[[2]int{h1, h2}] = true
    }
    return -1
}

func longestDupSubstring(s string) string {
    rand.Seed(time.Now().UnixNano())
    a1, a2 := randInt(26, 100), randInt(26, 100)
    //两个模，两个随机数，双重验证，减少哈希碰撞
    mod1, mod2 := randInt(1e9 + 7, math.MaxInt32), randInt(1e9 + 7, math.MaxInt32)

    arr := []byte(s)
    for i := range arr {
        arr[i] -= 'a'
    }

    l, r := 1, len(s) - 1
    length, start := 0, -1
    for l <= r {
        m := l + (r - l + 1) / 2
        idx := check(arr, m, a1, a2, mod1, mod2)
        if idx != -1 {
            l = m + 1
            length = m
            start = idx
        } else {
            r = m - 1
        }
    }
    if start == -1 {
        return ""
    }
    return s[start : start + length]
}
```

