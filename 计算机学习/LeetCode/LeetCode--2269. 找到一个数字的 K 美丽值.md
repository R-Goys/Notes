[2269. 找到一个数字的 K 美丽值](https://leetcode.cn/problems/find-the-k-beauty-of-a-number/)

> 一个整数 `num` 的 **k** 美丽值定义为 `num` 中符合以下条件的 **子字符串** 数目：
>
> - 子字符串长度为 `k` 。
> - 子字符串能整除 `num` 。
>
> 给你整数 `num` 和 `k` ，请你返回 `num` 的 k 美丽值。
>
> 注意：
>
> - 允许有 **前缀** **0** 。
> - `0` 不能整除任何值。
>
> 一个 **子字符串** 是一个字符串里的连续一段字符序列。

---

不要直接用数字来做，要先转成字符串，不然会变得不幸

```go
func divisorSubstrings(num int, k int) int {
    str := strconv.Itoa(num)
    curNum := 0
    ans := 0
    mul := math.Pow(10, float64(k - 1))
    for i, x := range str {
        curNum += int(x - '0')
        if i < k - 1 {
            curNum *= 10
            continue
        }
        if curNum != 0 && num % curNum == 0 {
            ans ++
        }
        curNum -= int(str[i - k + 1] - '0') * int(mul)
        curNum *= 10
    }
    return ans
}
```

