[8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/)

> 请你来实现一个 `myAtoi(string s)` 函数，使其能将字符串转换成一个 32 位有符号整数。
>
> 函数 `myAtoi(string s)` 的算法如下：
>
> 1. **空格：**读入字符串并丢弃无用的前导空格（`" "`）
> 2. **符号：**检查下一个字符（假设还未到字符末尾）为 `'-'` 还是 `'+'`。如果两者都不存在，则假定结果为正。
> 3. **转换：**通过跳过前置零来读取该整数，直到遇到非数字字符或到达字符串的结尾。如果没有读取数字，则结果为0。
> 4. **舍入：**如果整数数超过 32 位有符号整数范围 `[−231, 231 − 1]` ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 `−231` 的整数应该被舍入为 `−231` ，大于 `231 − 1` 的整数应该被舍入为 `231 − 1` 。
>
> 返回整数作为最终结果。

----

真是答辩题目，又长又臭，步骤：去除前导空格，拿到符号位，前导零，直接算，一旦超出范围，直接返回int32最大或最小值，一切正常则正常返回

```go
func myAtoi(s string) int {
	var sign int
	for len(s) > 0 && s[0] == ' ' {
		s = s[1:]
	}
	if len(s) > 0 && s[0] == '-' {
		sign = -1
	} else {
		sign = 1
	}
	if  len(s) > 0 && (s[0] == '+' || s[0] == '-') {
		s = s[1:]
	}
	for len(s) > 0 && s[0] == '0' {
		s = s[1:]
	}

	var ans int
	for _, num := range s {
		if num >= '0' && num <= '9' {
			ans = ans*10 + int(num-'0')
			if ans*sign >= math.MaxInt32 {
				return math.MaxInt32
			}
			if ans*sign <= math.MinInt32 {
				return math.MinInt32
			}
		} else {
			return ans * sign
		}
	}
	if ans*sign >= math.MaxInt32 {
		return math.MaxInt32
	}
	if ans*sign <= math.MinInt32 {
		return math.MinInt32
	}
	return ans * sign
}
```

