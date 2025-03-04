[43. 字符串相乘](https://leetcode.cn/problems/multiply-strings/)

> 给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。
>
> **注意：**不能使用任何内置的 BigInteger 库或直接将输入转换为整数。

----

字符串相乘，和字符串相加的思路差不多，这里我直接用利用数组来表示进位，而不利用切片，暂时我感觉这样更方便，由于N位数×M位数，最多只有N+M位数，所以按照这个性质来逐位相乘，然后考虑进位即可

```go
func multiply(num1 string, num2 string) string {
    if num1 == "0" || num2 == "0" {
		return "0"
	}
    n, m := len(num1), len(num2)
    ans := make([]int, m + n)
    for i := n - 1; i >= 0; i -- {
        x := n - i - 1
        for j := m - 1; j >= 0; j -- {
            y := m - j - 1
            ans[x + y] += int(num1[i] - '0') * int(num2[j] - '0')
            ans[x + y + 1] += ans[x + y] / 10
            ans[x + y] %= 10
        }
    }


    i := len(ans) - 1
    for ; i >= 0; i -- {
        if ans[i] == 0 {
            continue
        }
        break
    }

    var res string
    for ; i >= 0; i -- {
        res += strconv.Itoa(ans[i])
    }
    return res
}
```

