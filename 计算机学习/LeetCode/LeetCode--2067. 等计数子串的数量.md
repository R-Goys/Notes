[2067. 等计数子串的数量](https://leetcode.cn/problems/number-of-equal-count-substrings/)

> 给你一个下标从 **0** 开始的字符串 `s`，只包含小写英文字母和一个整数 `count`。如果 `s` 的 **子串** 中的每种字母在子串中恰好出现 `count` 次，这个子串就被称为 **等计数子串**。
>
> 返回 *`s` 中 **等计数子串** 的个数。*
>
> **子串** 是字符串中连续的非空字符序列。

---

前缀和+滑动窗口，这道题如果直接用之前的灵神那个题解 `O(N^2*26)` 会超时，为了减少重复计算，我们使用哈希表存储所有对应的前缀和，用于计算窗口值

```go
func equalCountSubstrings(s string, count int) int {
    src := make([][]int, len(s))
    for i := range src {
        src[i] = make([]int, 26)
    }
    ans := 0
    cur := make([]int, 26)
    for i, x := range s {
        cur[x - 'a'] ++

		if check(cur, count) {
			ans++
		}
        copy(src[i][:], cur[:])
        for j := 1; j <= 26 && i - j * count >= 0; j ++ {
			tmp := make([]int, 26)
			for k := 0; k < 26; k ++ {
	            tmp[k] = cur[k] - src[i - j * count][k]
			}
			if check(tmp, count) {
				ans ++
			}
        }
    }
    return ans
}

func check(src []int, cnt int) bool {
    for _, x := range src {
        if x != cnt && x != 0  {
            return false
        }
    }
    return true
}
```

