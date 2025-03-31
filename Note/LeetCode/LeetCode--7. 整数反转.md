[7. 整数反转](https://leetcode.cn/problems/reverse-integer/)

> 给你一个 32 位的有符号整数 `x` ，返回将 `x` 中的数字部分反转后的结果。
>
> 如果反转后整数超过 32 位的有符号整数的范围 `[−231, 231 − 1]` ，就返回 0。
>
> **假设环境不允许存储 64 位整数（有符号或无符号）。**

---

写完去看题解，发现我用的方法真几把low

```go
func reverse(x int) int {
    var sign int
    if x >= 0 {
        sign = 1
    } else {
        sign = -1
        x = -x
    }
    str := strconv.Itoa(x)
    bytes := []byte(str)
    for i, j := 0, len(bytes) - 1; i < j;{
        bytes[i], bytes[j] = bytes[j], bytes[i]
        i ++
        j -- 
    }
    str = string(bytes)
    num, _ := strconv.Atoi(str)
    if num * sign > math.MaxInt32 || num * sign < math.MinInt32 {
        return 0
    }

    return num * sign
}
```

空间优化

```go
func reverse(x int) int {
    n := 0
    for x != 0 {
        n = n * 10 + x % 10
        x /= 10
    }
    if n > math.MaxInt32 || n < math.MinInt32 {
        return 0
    }
    return n
}
```

