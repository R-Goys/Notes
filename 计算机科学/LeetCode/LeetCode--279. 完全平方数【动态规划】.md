[279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)

---

### 正文

#### 题目

给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

**完全平方数** 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。

**解释**：给定一个整数，要求将它分解为`a^2 + b^2 + c^2 + ...`的形式

----

#### 解决

抛开数学方法不提，这道题的突破口在于把整数 `n` 看成是 `n - x^2` 的形式，而这个数字必然小于 `n` 如此一来，便可以得出我们需要使用动态规划来解决，类似于背包问题，想到dp，这道题就比较简单了。

`i`每次+1，都需要根据之前的状态来更新，最后返回`f[n]`即可, 代码如下：

```go
func numSquares(n int) int {
    f := make([]int, n+1)
    //预处理可能用到的平方集合
    var nums []int
	for i := 1; i <= 10*n/i; i++ {
		nums = append(nums, i*i)
	}
    //计算
    for i := 1; i <= n; i++ {
        minn := 1000
        for j := 0; nums[j] <= i; j++ {
            minn = min(minn, f[i-nums[j]])
        }
        f[i] = minn + 1
    }
    return f[n]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

二刷，极简版。

```go
func numSquares(n int) int {
    src := make([]int, 0)
    for i := 0; i * i <= n; i ++{
        src = append(src, i * i)
    }
    m := len(src)
    f := make([]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = 0x3f3f3f3f
    }
    f[0] = 0
    for i := 0; i < m; i ++ {
        for j := src[i]; j <= n; j ++ {
            f[j] = min(f[j], f[j - src[i]] + 1)
        }
    }
    return f[n]
}
```

