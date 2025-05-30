[1151. 最少交换次数来组合所有的 1](https://leetcode.cn/problems/minimum-swaps-to-group-all-1s-together/)

> 给出一个二进制数组 `data`，你需要通过交换位置，将数组中 **任何位置** 上的 1 组合到一起，并返回所有可能中所需 **最少的交换次数**。

---

初见，还以为有多难，其实只需要算出一共有多少个 1，然后以 1 的总数为窗口去计算 1 的最大数量就行了，而最后的答案就是 k - ans：

```go
func minSwaps(data []int) int {
    k := 0
    sum := 0
    ans := 0
    for _, x := range data {
        k += x
    }
    if k == 0 {
        return 0
    }
    for i, x := range data {
        sum += x
        if i < k - 1 {
            continue
        }
        ans = max(sum, ans)
        sum -= data[i - k + 1]
    }
    return k - ans
}
```

