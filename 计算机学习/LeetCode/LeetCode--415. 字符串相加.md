[415. 字符串相加](https://leetcode.cn/problems/add-strings/)

> 给定两个字符串形式的非负整数 `num1` 和`num2` ，计算它们的和并同样以字符串形式返回。
>
> 你不能使用任何內建的用于处理大整数的库（比如 `BigInteger`）， 也不能直接将输入的字符串转换为整数形式。

---

### 高精度加法

```go
func addStrings(num1 string, num2 string) string {
    var NumA, NumB []int
    for i := len(num1) - 1; i >= 0; i -- {
        NumA = append(NumA, int(num1[i] - '0'))
    }
    for i := len(num2) - 1; i >= 0; i -- {
        NumB = append(NumB, int(num2[i] - '0'))
    }
    ans := Eval(NumA, NumB)
    StringAns := ""
    for i := len(ans) - 1; i >= 0; i -- {
        StringAns += string(ans[i] + '0')
    }
    return StringAns
}

func Eval(NumA []int, NumB []int) []int {
    var ans []int
    temp := 0
    i := 0
    for ; i < len(NumA) && i < len(NumB); i ++ {
        ans = append(ans, (NumA[i] + NumB[i] + temp) % 10)
        temp = (NumA[i] + NumB[i] + temp) / 10
    }
    for len(NumA) > i {
        ans = append(ans, ((NumA[i]) + temp) % 10)
        temp = (NumA[i] + temp) / 10
        i ++
    }
    for len(NumB) > i {
        ans = append(ans, ((NumB[i]) + temp) % 10)
        temp = (NumB[i] + temp) / 10
        i ++
    }
    for temp != 0 {
        ans = append(ans, temp)
        temp /= 10
    }
    return ans
}
```

---

### 官方做法

看了题解感觉自己好蠢...不过也算复习了一下高精度加法了

```go
func addStrings(num1 string, num2 string) string {
    temp := 0
    ans := ""
    for i, j := len(num1) - 1, len(num2) - 1; i >= 0 || j >= 0 || temp != 0; i, j = i - 1, j - 1 {
        var a, b int
        if i >= 0 {
            a = int(num1[i] - '0')
        }
        if j >= 0 {
            b = int(num2[j] - '0')
        }
        temp += a + b
        ans = strconv.Itoa(temp % 10) + ans
        temp /= 10
    }
    return ans
}
```

