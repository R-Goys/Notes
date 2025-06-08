[3577. 统计计算机解锁顺序排列数](https://leetcode.cn/problems/count-the-number-of-computer-unlocking-permutations/)

> 给你一个长度为 `n` 的数组 `complexity`。
>
> 在房间里有 `n` 台 **上锁的** 计算机，这些计算机的编号为 0 到 `n - 1`，每台计算机都有一个 **唯一** 的密码。编号为 `i` 的计算机的密码复杂度为 `complexity[i]`。
>
> 编号为 0 的计算机密码已经 **解锁** ，并作为根节点。其他所有计算机必须通过它或其他已经解锁的计算机来解锁，具体规则如下：
>
> - 可以使用编号为 `j` 的计算机的密码解锁编号为 `i` 的计算机，其中 `j` 是任何小于 `i` 的整数，且满足 `complexity[j] < complexity[i]`（即 `j < i` 并且 `complexity[j] < complexity[i]`）。
> - 要解锁编号为 `i` 的计算机，你需要事先解锁一个编号为 `j` 的计算机，满足 `j < i` 并且 `complexity[j] < complexity[i]`。
>
> 求共有多少种 `[0, 1, 2, ..., (n - 1)]` 的排列方式，能够表示从编号为 0 的计算机（唯一初始解锁的计算机）开始解锁所有计算机的有效顺序。
>
> 由于答案可能很大，返回结果需要对 **109 + 7** 取余数。
>
> **注意：**编号为 0 的计算机的密码已解锁，而 **不是** 排列中第一个位置的计算机密码已解锁。
>
> **排列** 是一个数组中所有元素的重新排列。

---

真是脑筋急转弯，一开始还想着用暴搜和 dp，然后没有写出来，看了 0 神的题解真是太厉害了😭😭

可以直接用 `n!` 的原因是可以想象一下，我们这里是求排列，事实上，我们的用 0 先解锁 1 再解锁 2，其实和用 0 解锁 1 再用 1 解锁 2 的排列是一样的，所以用 dfs 太傻瓜了。

```go
func countPermutations(complexity []int) int {
    mod := 1000000000 + 7
    ans := 1
    for i := 1; i < len(complexity); i ++ {
        if complexity[i] <= complexity[0] {
            return 0
        }
        ans = ans * i % mod
    }
    return ans
}
```

