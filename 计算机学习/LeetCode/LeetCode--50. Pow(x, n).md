[50. Pow(x, n)](https://leetcode.cn/problems/powx-n/)

> 实现 [pow(*x*, *n*)](https://www.cplusplus.com/reference/valarray/pow/) ，即计算 `x` 的整数 `n` 次幂函数（即，`xn` ）。

之前压根没听过快速幂的算法，今天开眼了，**递归**

```go
func myPow(x float64, n int) float64 {
    if n >= 0 {
        return quickMul(x, n)
    } 
    return 1.0 / quickMul(x, -n)
}

func quickMul(x float64, n int) float64 {
    if n == 0 {
        return 1.0
    }
    y := quickMul(x, n/2)
    if n % 2 == 0 {
        return y * y
    }
    return y * y * x
}
```

**迭代**

```go
func myPow(x float64, n int) float64 {
    if n >= 0 {
        return quickMul(x, n)
    } 
    return 1.0 / quickMul(x, -n)
}

func quickMul(x float64, n int) float64 {
    var ans float64 = 1.0
    var x_contribute float64 = x
    for n > 0 {
        if n % 2 == 1 {
            ans *= x_contribute
        }
        x_contribute *= x_contribute
        n /= 2
    }
    return ans
}
```

