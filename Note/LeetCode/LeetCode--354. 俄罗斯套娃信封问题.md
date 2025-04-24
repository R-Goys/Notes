[354. 俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)

> 给你一个二维整数数组 `envelopes` ，其中 `envelopes[i] = [wi, hi]` ，表示第 `i` 个信封的宽度和高度。
>
> 当另一个信封的宽度和高度都比这个信封大的时候，这个信封就可以放进另一个信封里，如同俄罗斯套娃一样。
>
> 请计算 **最多能有多少个** 信封能组成一组“俄罗斯套娃”信封（即可以把一个信封放到另一个信封里面）。
>
> **注意**：不允许旋转信封。

---

第一种方法，排序，嵌套循环，遍历最后一层的信封，计算出以每一层信封结尾的信封数量，优点是简单易懂，缺点是超时了。。

```go
func maxEnvelopes(envelopes [][]int) int {
    n := len(envelopes)
    ans := 1
    sort.Slice(envelopes, func(i, j int) bool {
        return envelopes[i][0] < envelopes[j][0] ||
        (envelopes[i][0] == envelopes[j][0] && envelopes[i][1] > envelopes[j][1])
    })
    f := make([]int, n)
    for i := 0; i < n; i ++ {
        f[i] = 1
    }
    for i := 1; i < n; i ++ {
        for j := 0; j < i ; j ++ {
            if envelopes[j][1] < envelopes[i][1] {
                f[i] = max(f[i], f[j] + 1)
                ans = max(f[i], ans)
            }
        }
    }
    return ans
}
```

另一种方法是二分查找，和之前坐过的最长对数链的思路类似。

排序之后，一层遍历每个信封，此时第一个数字都是有序的，我们只需要考虑第二个数字，在我们的dp切片中二分查找我们需要插入的位置，这个位置是最后一个小于被搜索的数字的元素的下一个位置，如果发现这个位置大于了len，那么就需要新加入元素，如果不是，就直接覆盖这个数字即可。

```go
func maxEnvelopes(envelopes [][]int) int {
    n := len(envelopes)
    sort.Slice(envelopes, func(i, j int) bool {
        return envelopes[i][0] < envelopes[j][0] ||
        (envelopes[i][0] == envelopes[j][0] && envelopes[i][1] > envelopes[j][1])
    })
    f := make([]int, 0)
    for i := 0; i < n; i ++ {
        if dst := sort.SearchInts(f, envelopes[i][1]); dst < len(f) {
            f[dst] = envelopes[i][1]
        } else {
            f = append(f, envelopes[i][1])
        }
    }
    return len(f)
}
```

