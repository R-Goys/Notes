[1852. 每个子数组的数字种类数](https://leetcode.cn/problems/distinct-numbers-in-each-subarray/)

> 给你一个长度为 `n` 的整数数组 `nums` 与一个整数 `k`。你的任务是找到 `nums` **所有** 长度为 `k` 的子数组中 **不同** 元素的数量。
>
> 返回一个数组 `ans`，其中 `ans[i]` 是对于每个索引 `0 <= i < n - k`，`nums[i..(i + k - 1)]` 中不同元素的数量。

---

一个套路，不得不说灵神题单真厉害，这几个题目全是一个套路，但是都有各自的写法，感觉对记忆模板挺有帮助的。

```go
func distinctNumbers(nums []int, k int) []int {
    ans := make([]int, 0)
    mp := make(map[int]int, 0)
    sum := 0
    for i, x := range nums {
        if mp[x] == 0 {
            sum ++
        }
        mp[x] ++
        if i < k - 1 {
            continue
        }
        ans = append(ans, sum)
        if mp[nums[i - k + 1]] == 1 {
            sum --
        }
        mp[nums[i - k + 1]] --
    }
    return ans
}
```

