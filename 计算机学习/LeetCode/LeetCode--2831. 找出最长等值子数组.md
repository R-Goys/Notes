[2831. 找出最长等值子数组](https://leetcode.cn/problems/find-the-longest-equal-subarray/)

> 给你一个下标从 **0** 开始的整数数组 `nums` 和一个整数 `k` 。
>
> 如果子数组中所有元素都相等，则认为子数组是一个 **等值子数组** 。注意，空数组是 **等值子数组** 。
>
> 从 `nums` 中删除最多 `k` 个元素后，返回可能的最长等值子数组的长度。
>
> **子数组** 是数组中一个连续且可能为空的元素序列。

---

第一次没写出来，这里使用分组滑窗，每个窗口对应一组相同的数字，我们需要用数组存储这组数字之前需要删除多少个数字的前缀，然后利用这个来进行分组滑窗

```go
func longestEqualSubarray(nums []int, k int) int {
    posLists := make([][]int, len(nums) + 1)
    ans := 0
    for i, x := range nums {
        posLists[x] = append(posLists[x], i - len(posLists[x]))
    }
    for _, pos := range posLists {
        if len(pos) <= ans {
            continue
        }
        left := 0
        for right, p := range pos {
            for p - pos[left] > k {
                left ++
            }
            ans = max(ans, right - left + 1)
        }
    }
    return ans
}
```

