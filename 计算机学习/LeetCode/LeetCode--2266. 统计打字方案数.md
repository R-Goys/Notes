[2266. 统计打字方案数](https://leetcode.cn/problems/count-number-of-texts/)

> Alice 在给 Bob 用手机打字。数字到字母的 **对应** 如下图所示。
>
> ![img](https://pic.leetcode.cn/1722224025-gsUAIv-image.png)
>
> 为了 **打出** 一个字母，Alice 需要 **按** 对应字母 `i` 次，`i` 是该字母在这个按键上所处的位置。
>
> - 比方说，为了按出字母 `'s'` ，Alice 需要按 `'7'` 四次。类似的， Alice 需要按 `'5'` 两次得到字母 `'k'` 。
> - 注意，数字 `'0'` 和 `'1'` 不映射到任何字母，所以 Alice **不** 使用它们。
>
> 但是，由于传输的错误，Bob 没有收到 Alice 打字的字母信息，反而收到了 **按键的字符串信息** 。
>
> - 比方说，Alice 发出的信息为 `"bob"` ，Bob 将收到字符串 `"2266622"` 。
>
> 给你一个字符串 `pressedKeys` ，表示 Bob 收到的字符串，请你返回 Alice **总共可能发出多少种文字信息** 。
>
> 由于答案可能很大，将它对 `10^9 + 7` **取余** 后返回。

---

不会，为了写出这道题，我们需要将长度为 n 的连续数字可以组合成的组合总和计算出来，这里也是动态规划的思想，相当于第一次 dp ，随后，我们根据计算出来的 length-num 的数组进行计算，我们遍历字符串，记录重复子串的长度，遇到不相同的字符的时候，就直接根据之前 dp 的数组进行乘积：

```go
func countTexts(pressedKeys string) int {
    mod := 1000000007
    n := len(pressedKeys)
    // 先将不同字符串长度可以拆分成的组合数统计出来。
    dp3 := []int{1, 1, 2, 4}
    dp4 := []int{1, 1, 2, 4}
    for i := 4; i <= n; i ++ {
        dp3 = append(dp3, (dp3[i - 1] + dp3[i - 2] + dp3[i - 3]) % mod)
        dp4 = append(dp4, (dp4[i - 1] + dp4[i - 2] + dp4[i - 3] + dp4[i - 4]) % mod)
    }
    ans := 1
    cnt := 1
    for i := 1; i < n; i ++ {
        if pressedKeys[i] == pressedKeys[i - 1] {
            cnt ++
        } else {
            if pressedKeys[i - 1] == '7' || pressedKeys[i - 1] == '9' {
                ans = (ans * dp4[cnt]) % mod
            } else {
                ans = (ans * dp3[cnt]) % mod
            }
            cnt = 1
        }
    }
    // 最后一组串还没有统计
    if pressedKeys[n - 1] == '7' || pressedKeys[n - 1] == '9' {
        ans = (ans * dp4[cnt]) % mod
    } else {
        ans = (ans * dp3[cnt]) % mod
    }
    return ans
}
```

