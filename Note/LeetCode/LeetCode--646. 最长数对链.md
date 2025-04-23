[646. 最长数对链](https://leetcode.cn/problems/maximum-length-of-pair-chain/)

> 给你一个由 `n` 个数对组成的数对数组 `pairs` ，其中 `pairs[i] = [lefti, righti]` 且 `lefti < righti` 。
>
> 现在，我们定义一种 **跟随** 关系，当且仅当 `b < c` 时，数对 `p2 = [c, d]` 才可以跟在 `p1 = [a, b]` 后面。我们用这种形式来构造 **数对链** 。
>
> 找出并返回能够形成的 **最长数对链的长度** 。
>
> 你不需要用到所有的数对，你可以以任何顺序选择其中的一些数对来构造。

---

这道题方法很多，最简单的一种就是贪心，按照第二个数字排序，然后依次比对第一个数字是否满足要求，在这种情况下，可以最大程度的满足我们希望让追加在后面的数字尽可能小，所以这样的算法是可行的。

```go
func findLongestChain(pairs [][]int) int {
    sort.Slice(pairs, func(i, j int) bool {
        return pairs[i][1] < pairs[j][1]
    })
    cur := -114514
    ans := 0
    for i := 0; i < len(pairs); i ++ {
        if cur < pairs[i][0] {
            cur = pairs[i][1]
            ans ++
        }
    }
    return ans
}
```

第二种是按照第一个数字排序，然后两层for遍历，性能上不如上一种，但是逻辑上更清晰一点，容易理解一点。

```go
func findLongestChain(pairs [][]int) int {
    sort.Slice(pairs, func(i, j int) bool {
        return pairs[i][0] < pairs[j][0]
    })
    n := len(pairs)
    f := make([]int, n + 1)

    for i := 0; i < n; i ++ {
        f[i] = 1
        for j := 0; j < i; j ++ {
            if pairs[i][0] > pairs[j][1] {
                f[i] = max(f[i], f[j] + 1)
            }
        }    
    }
    return f[n - 1]
}
```

第三种，和之前做过的最长上升子序列的二分查找的方法很类似，如果查找的数字超出了范围，则追加在尾部，如果在范围之内，则覆盖这个数字，这样，无论如何，我们最终得到的长度都会是正确的结果。

```go
func findLongestChain(pairs [][]int) int {
    sort.Slice(pairs, func(i, j int) bool {
        return pairs[i][0] < pairs[j][0]
    })
    ans := make([]int, 0)
    for i := 0; i < len(pairs); i ++ {
        l, r := 0, len(ans)
        // 二分查找第一个比他小的数字。
        for l < r {
            mid := (l + r) / 2
            if ans[mid] < pairs[i][0] {
                l = mid + 1
            } else {
                r = mid
            }
        }
        if l < len(ans) {
            ans[l] = min(ans[l], pairs[i][1]) 
        } else {
            ans = append(ans, pairs[i][1])
        }
    }
    return len(ans)
}
```

