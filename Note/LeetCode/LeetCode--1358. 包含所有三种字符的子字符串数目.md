[1358. 包含所有三种字符的子字符串数目](https://leetcode.cn/problems/number-of-substrings-containing-all-three-characters/)

> 给你一个字符串 `s` ，它只包含三种字符 a, b 和 c 。
>
> 请你返回 a，b 和 c 都 **至少** 出现过一次的子字符串数目。

---

数组存储每个数字出现的次数，由于对应包含符合条件的子串的字符串都是符合条件的，所以此处需要计算 right 右边的父串长度。

然后依旧滑动窗口

```go
func numberOfSubstrings(s string) int {
    ans := 0
    cnt :=  ['z']int{}
    n := len(s)
    num := 3
    l := 0
    for r, x := range s {
        if cnt[x] == 0 {
            num --
        }
        cnt[x] ++
        for num == 0 {
            ans += n - r
            cnt[s[l]] --
            if cnt[s[l]] == 0 {
                num ++
            }
            l ++
        }
    }
    return ans
}
```

