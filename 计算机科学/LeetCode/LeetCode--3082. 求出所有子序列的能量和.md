[3082. 求出所有子序列的能量和](https://leetcode.cn/problems/find-the-sum-of-the-power-of-all-subsequences/)

> 给你一个长度为 `n` 的整数数组 `nums` 和一个 **正** 整数 `k` 。
>
> 一个整数数组的 **能量** 定义为和 **等于** `k` 的子序列的数目。
>
> 请你返回 `nums` 中所有子序列的 **能量和** 。
>
> 由于答案可能很大，请你将它对 `10^9 + 7` **取余** 后返回。

---

先统计长度为 i 的和为 k 的子序列的数量，然后我们可以计算出有多少个子序列可以包含他们，这里就是数学问题了，看了 0 神的我才知道有 `2^(n - c)` 种，然后就可以做了：

```go
func sumOfPower(nums []int, k int) int {
    n := len(nums)
    ans := 0
    mod := 1000000000 + 7
    f := make([][]int, k + 1)
    for i := 0; i <= k; i ++ {
        f[i] = make([]int, n + 1)
    }
    f[0][0] = 1
    for i, num := range nums {
        for j := k; j >= num; j -- {
            for c := i + 1; c > 0; c -- {
                f[j][c] = (f[j][c] + f[j - num][c - 1]) % mod
            }
        }
    }
    pow2 := 1
    for i := n; i > 0; i -- {
        ans = (ans + f[k][i] * pow2) % mod
        pow2 = pow2 * 2 % mod
    }
    return ans
}
```

另一种方法，不需要使用 pow2，看起来比较优雅，但是有点抽象，理解起来很有难度，每一轮遍历， i 相当于是对应的序列长度，而 j 代表着对应的子序列的和。

在这里的一维数组就需要自行乘二来进行状态转移：

```go
func sumOfPower(nums []int, k int) int {
    mod := 1000000000 + 7
    n := len(nums)
    f := make([]int, k + 1)
    f[0] = 1
    for i := 0; i < n; i ++ {
        for j := k; j >= 0; j -- {
            if j >= nums[i] {
                f[j] = (f[j] * 2 + f[j - nums[i]]) % mod
            } else {
                f[j] = f[j] * 2 % mod
            }
        }
    } 
    return f[k]
}
```

