[5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

----

>给你一个字符串 `s`，找到 `s` 中最长的回文子串。

## 中心扩展

这道题虽然可以采取暴力枚举的方式来做，但是O(n^3)，这道题比较直观的做法就是中心扩展法，遍历字符串，以每一个字符为中心扩展，依次下来取得长度最大的长度，用常数级的遍历存储最大子串的左右边界，代码如下：

```go
func expandAroundCenter(s string, left, right int) (int, int) {
	for left >= 0 && right < len(s) && s[left] == s[right] {
		left--
		right++
	}
	return left + 1, right - 1
}

func longestPalindrome(s string) string {
	start, end := 0, 0
	for i := 0; i < len(s); i++ {
		left1, right1 := expandAroundCenter(s, i, i)
		left2, right2 := expandAroundCenter(s, i, i+1) 
		if right1-left1 > end-start {
			start = left1
			end = right1
		}
		if right2-left2 > end-start {
			start = left2
			end = right2
		}
	}
	return s[start:end+1]
}
```

## DP

当然，这道题也算常规的动态规划类题目，也比较直观，枚举子串的左右边界，根据前一个状态判断下一个状态，这里需要注意的是前一个状态如何表示。

比如一个回文子串`s[L:R]`，长度大于3，如果需要判断它是否为回文子串，那么就需要知道`s[L + 1: R - 1]`的状态，因此我们需要注意遍历的顺序,代码如下：

```go
func longestPalindrome(s string) string {
    n := len(s)
    
    L := 0
    R := 1
    f := make([][]bool, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]bool, n)
        f[i][i] = true
    }
    for i := 1; i < n; i ++ {
        for j := i - 1; j >= 0; j -- {
            if s[i] != s[j] {
                f[i][j] = false
            } else {
                if i - j <= 2 {
                    f[i][j] = true
                } else {
                    f[i][j] = f[i - 1][j + 1]
                }
            }

            if f[i][j] && i - j + 1 > R - L {
                L = j
                R = i + 1
            }
        }
    }
    return s[L:R]
}
```

---

二刷，ok

```go
func longestPalindrome(s string) string {
    n := len(s)
    f := make([][]bool, n)
    for i := 0; i < n; i ++ {
        f[i] = make([]bool, n)
        f[i][i] = true
    }
    MaxL, MaxR := 0, 1
    for i := 1; i < n; i ++ {
        for j := i - 1; j >= 0; j -- {
            if s[i] == s[j] {
                if i == j + 1 {
                    f[i][j] = true
                } else {
                    f[i][j] = f[i - 1][j + 1]
                }
                if i - j + 1 > MaxR - MaxL && f[i][j] {
                    MaxL, MaxR = j, i + 1
                }
            }
        }
    }
    return s[MaxL: MaxR]
}
```

