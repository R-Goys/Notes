[1004. 最大连续1的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/)

> 给定一个二进制数组 `nums` 和一个整数 `k`，假设最多可以翻转 `k` 个 `0` ，则返回执行操作后 *数组中连续 `1` 的最大个数* 。

---

一开始还以为是动态规划，反应过来滑动窗口更方便。

```go
func longestOnes(nums []int, k int) int {
    n := len(nums)
    ans := 0
    for i, j := 0, 0; j < n; j ++ {
        k -= 1 - nums[j]
        for k < 0 {
            k += 1 - nums[i]
            i ++
        }
        ans = max(ans, j - i + 1)
    }
    return ans
}
```

