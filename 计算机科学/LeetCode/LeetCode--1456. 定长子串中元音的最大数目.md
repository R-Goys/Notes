[1456. 定长子串中元音的最大数目](https://leetcode.cn/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)

> 给你字符串 `s` 和整数 `k` 。
>
> 请返回字符串 `s` 中长度为 `k` 的单个子字符串中可能包含的最大元音字母数。
>
> 英文中的 **元音字母** 为（`a`, `e`, `i`, `o`, `u`）。

---

老老实实从头开始刷 0 神题单了，dp 刷了很多，感觉提升不大。

经典方法：

```go
func maxVowels(s string, k int) int {
    mp := make(map[byte]int, 0)
    mp['a'], mp['e'], mp['i'], mp['o'], mp['u'] = 1, 1, 1, 1, 1
    l, r := 0, 0
    sum := 0
    ans := 0
    n := len(s)
    for r < n {
        if r - l < k && r < n {
            sum += mp[s[r]]
            r ++
            ans = max(ans, sum)
        }
        if r - l == k {
            sum -= mp[s[l]]
            l ++
        }
    }
    return ans
}
```

另一种方法，直接在循环遍历的时候判断，也是一个好方法，参考 0 神的。

```go
func maxVowels(s string, k int) (ans int) {
    vowel := 0
    for i, in := range s {
        if in == 'a' || in == 'e' || in == 'i' || in == 'o' || in == 'u' {
            vowel ++
        }
        if i < k - 1 { 
            continue
        }

        ans = max(ans, vowel)
        
        out := s[i - k + 1]
        if out == 'a' || out == 'e' || out == 'i' || out == 'o' || out == 'u' {
            vowel --
        }
    }
    return
}
```

