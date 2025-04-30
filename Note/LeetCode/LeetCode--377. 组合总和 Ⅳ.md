[377. 组合总和 Ⅳ](https://leetcode.cn/problems/combination-sum-iv/)

> 给你一个由 **不同** 整数组成的数组 `nums` ，和一个目标整数 `target` 。请你从 `nums` 中找出并返回总和为 `target` 的元素组合的个数。
>
> 题目数据保证答案符合 32 位整数范围。

---

最开始还以为是求组合的数组，想用 dfs ，然后发现求组合数量就行了，一眼背包，光速解决。

```go
func combinationSum4(nums []int, target int) int {
    f := make([]int, target + 1)
    n := len(nums)
    f[0] = 1
    for i := 0; i <= target; i ++ {
        for j := 0; j < n; j ++ {
            if nums[j] <= i {
                f[i] += f[i - nums[j]];
            }
        }
    }
    return f[target]
}
```

二刷

```go
func combinationSum4(nums []int, target int) int {
    n := len(nums)
    f := make([]int, target + 1)
    f[0] = 1
    for v := 1; v <= target; v ++ {
        for i := 0; i < n; i ++ {
            if nums[i] <= v {
                f[v] += f[v - nums[i]]
            }
        }
    }
    return f[target]
}
```

