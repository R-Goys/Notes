[87. 扰乱字符串](https://leetcode.cn/problems/scramble-string/)

> 使用下面描述的算法可以扰乱字符串 `s` 得到字符串 `t` ：
>
> 1. 如果字符串的长度为 1 ，算法停止
> 2. 如果字符串的长度 > 1 ，执行下述步骤：
>    - 在一个随机下标处将字符串分割成两个非空的子字符串。即，如果已知字符串 `s` ，则可以将其分成两个子字符串 `x` 和 `y` ，且满足 `s = x + y` 。
>    - **随机** 决定是要「交换两个子字符串」还是要「保持这两个子字符串的顺序不变」。即，在执行这一步骤之后，`s` 可能是 `s = x + y` 或者 `s = y + x` 。
>    - 在 `x` 和 `y` 这两个子字符串上继续从步骤 1 开始递归执行此算法。
>
> 给你两个 **长度相等** 的字符串 `s1` 和 `s2`，判断 `s2` 是否是 `s1` 的扰乱字符串。如果是，返回 `true` ；否则，返回 `false` 。

---

这道题完全就是按照题解来的，说是dp，而很大的篇幅都是dfs，虽然最开始想过用dfs，但是不知道dp + dfs如何下手，之前从来没有试过这样玩。

官解还是比较清晰的，主要的思路就是：

1. 构造状态转移的数组

2. 构造dfs函数，这个函数会传入三个参数，字符串1的起始位置，字符串2的起始位置以及需要判断的两个字符串的长度length，如何判断？

3. 最开始，先判断两个字符串是否完全一样，如果一样，直接返回1，

4. 下一步则是再判断两个字符串的对应的字母数量是否完全对等，如果不是，返回0，如果完全对等，说明还有可能是扰乱字符串，继续下一步。

5. 这一步就是递归到下一层去判断了，这里需要枚举分割的位置，所以消耗的空间很大，而这里还需要进行两种判断，一种是交换两个子串的位置的情况，一种是不交换，直接分割。

6. 随后就可以返回我们的结果了。

```go
func isScramble(s1 string, s2 string) bool {
    n := len(s1)
    f := make([][][]int8, n)
    for i := 0; i < n; i ++ {
        f[i] = make([][]int8, n)
        for j := 0; j < n; j ++ {
            f[i][j] = make([]int8, n + 1)
            for k := 0; k <= n; k ++ {
                f[i][j][k] = -1
            }
        }
    }
    var dfs func(i1, i2, length int) int8
    dfs = func(i1, i2, length int) (res int8) {
        d := &f[i1][i2][length]
        if *d != -1 {
            return *d
        }
        defer func() {
            *d = res
        }()

        x, y := s1[i1 : i1 + length], s2[i2 : i2 + length]
        if x == y {
            return 1
        }

        freq := [26]int{}
        for i, ch := range x {
            freq[ch - 'a'] ++
            freq[y[i] - 'a'] --
        }
        for _, f := range freq[:] {
            if f != 0 {
                return 0
            }
        }
        // 枚举分割位置
        for i := 1; i < length; i ++ {
            if dfs(i1, i2, i) == 1 && dfs(i1 + i, i2 + i, length - i) == 1 {
                return 1
            }
            //交换字符串的情况
            if dfs(i1, i2 + length - i, i) == 1 && dfs(i1 + i, i2, length - i) == 1 {
                return 1
            }
        }
        return 0
    }
    return dfs(0, 0, n) == 1
}
```

