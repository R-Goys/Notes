[440. 字典序的第K小数字](https://leetcode.cn/problems/k-th-smallest-in-lexicographical-order/)

> 给定整数 `n` 和 `k`，返回 `[1, n]` 中字典序第 `k` 小的数字。

---

Hard，战斗爽，依然是看题解，在最开始没看懂另一个函数是啥意思，很久才反应过来原来是计算以当前数字为前缀的并且数值小于n的总个数，然后就反应过来这道题是怎么写的了。

首先最开始的prefix是1，然后以其为前缀，看看k是否存在于以k为前缀的数字中，如果存在，即`cnt > k`，所以此时继续深入到以该prefix为前缀的下一层前缀，就是增大他的前缀，以便搜索，简单理解为树的搜索就行。

如果当前不存在，即`cnt <= k`，那么就prefix++，进入到下一个分支看看存不存在k。

```go
func findKthNumber(n int, k int) int {
    prefix := 1
    k --
    for k > 0 {
        cnt := getPrefixCount(prefix, n)
        if cnt <= k {
            k -= cnt
            prefix ++
        } else {
            prefix *= 10
            k --
        }
    }
    return prefix
}

func getPrefixCount(prefix, n int) int {
    first, last := prefix, prefix
    cnt := 0
    for first <= n {
        cnt += min(last, n) - first + 1
        first *= 10
        last = last * 10 + 9
    }
    return cnt
}
```

