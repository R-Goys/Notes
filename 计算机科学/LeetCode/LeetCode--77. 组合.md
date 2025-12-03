[77. 组合](https://leetcode.cn/problems/combinations/)

> 给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。
>
> 你可以按 **任何顺序** 返回答案。

---

大物学累了，真的无聊，写一下算法题放松一下~

方法一，传统dfs搜索，由于遍历的所有情况均是正确的，故无需剪枝

```go
func combine(n int, k int) [][]int {
    ans := [][]int{}
    dfs(&ans, 0, k, n, []int{}, 1)
    return ans
}

func dfs(ans *[][]int, curLen, maxLen, maxNum int, cur []int, st int) {
    if curLen == maxLen {
        *ans = append(*ans, append([]int{}, cur...))
        return
    }

    for i := st; i <= maxNum; i ++ {
        cur = append(cur, i)
        dfs(ans, curLen + 1, maxLen, maxNum, cur, i + 1)
        cur = cur[:len(cur) - 1]
    }
}
```

方法二，原本感觉官方这个不用递归的方法还挺高级的，跑出来空间复杂度比我递归好高。。。

这个方法光看代码，反正我是无法理解的，但是在草稿纸上面画个图，你就明白是怎么回事了，本质上来讲，其实和官方题解上面说的二进制的思想是大差不差的，不过讲得有点不是人话。

每一次循环，都会去令一个数字+1，保证当时的组合不同，由于是组合，所以也没有必要考虑顺序，可以想成是每一次的tmp[j] ++，都是让我们的二进制组合的对应的一个数字移动到下一个位置，这样，就可以保证遍历完所有的组合。

其实自己画一下是最好的，不然可能也不知道我在说什么:)

```go
func combine(n int, k int) [][]int {
    tmp := make([]int, k)
    for i := 0; i < k; i ++ {
        tmp[i] = i + 1
    }
    tmp = append(tmp, n + 1)
    ans := [][]int{}
    for j := 0; j < k; {
        ans = append(ans, append([]int{}, tmp[:k]...))
        for j = 0; j < k && tmp[j] + 1 == tmp[j + 1]; j ++ {
            tmp[j] = j + 1
        }
        tmp[j] ++
    }
    return ans
}
```

---

感觉最近乱刷题刷多了，对于专题部分却没有什么突破，接下来还是搁置codetop了，专注于一个专题训练了！