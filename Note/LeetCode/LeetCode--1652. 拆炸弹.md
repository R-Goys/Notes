[1652. 拆炸弹](https://leetcode.cn/problems/defuse-the-bomb/)

> 你有一个炸弹需要拆除，时间紧迫！你的情报员会给你一个长度为 `n` 的 **循环** 数组 `code` 以及一个密钥 `k` 。
>
> 为了获得正确的密码，你需要替换掉每一个数字。所有数字会 **同时** 被替换。
>
> - 如果 `k > 0` ，将第 `i` 个数字用 **接下来** `k` 个数字之和替换。
> - 如果 `k < 0` ，将第 `i` 个数字用 **之前** `k` 个数字之和替换。
> - 如果 `k == 0` ，将第 `i` 个数字用 `0` 替换。
>
> 由于 `code` 是循环的， `code[n-1]` 下一个元素是 `code[0]` ，且 `code[0]` 前一个元素是 `code[n-1]` 。
>
> 给你 **循环** 数组 `code` 和整数密钥 `k` ，请你返回解密后的结果来拆除炸弹！

---

挺简单的，就是条件挺多，要考虑一下边界问题：

```go
func decrypt(code []int, k int) []int {
    n := len(code)
    sum := 0
    ans := make([]int, n)
    if k == 0 {
        return ans
    }
    if k > 0 {
        for i := 0; i < n + k; i ++ {
            sum += code[i % n]
            if i < k - 1 {
                continue
            }
            ans[max(0, i - k)] = sum
            if i - k + 1 < n {
                sum -= code[i - k + 1]
            }
        }
    } else {
        k = -k
        for i := 0; i < n + k; i ++ {
            sum += code[i % n]
            if i < k - 1 {
                continue
            }
            ans[(i + 1) % n] = sum
            if i - k + 1 < n {
                sum -= code[i - k + 1]
            }
        }
    }
    return ans
}
```

