[1461. 检查一个字符串是否包含所有长度为 K 的二进制子串](https://leetcode.cn/problems/check-if-a-string-contains-all-binary-codes-of-size-k/)

> 给你一个二进制字符串 `s` 和一个整数 `k` 。如果所有长度为 `k` 的二进制字符串都是 `s` 的子串，请返回 `true` ，否则请返回 `false` 。

---

暴力出奇迹

```go
func hasAllCodes(s string, k int) bool {
    mp := make(map[string]int)
    reqLen := math.Pow(2, float64(k))
    for i := range s {
        if i < k - 1 {
            continue
        }
        mp[s[i - k + 1 : i + 1]] ++
    }
    return len(mp) == int(reqLen)
}
```

优化方案，省去了截取字符串带来的消耗，直接在数字的基础上加减。

```go
func hasAllCodes(s string, k int) bool {
    if len(s) < k {
        return false
    }
	mp := make(map[int]int)
	cur := 0
	for i := 0; i < len(s) && i < k; i ++ {
		cur <<= 1
		cur += int(s[i] - '0')
	}
	reqLen := int(math.Pow(2, float64(k)))
	mask := int(math.Pow(2, float64(k))) - 1
	mp[cur]++
	for i := k; i < len(s); i++ {
		cur <<= 1
		cur &= mask
		cur += int(s[i] - '0')
		mp[cur]++
	}
	return len(mp) == int(reqLen)
}
```

