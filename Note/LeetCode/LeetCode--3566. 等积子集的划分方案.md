[3566. 等积子集的划分方案](https://leetcode.cn/problems/partition-array-into-two-equal-product-subsets/)

> 给你一个整数数组 `nums`，其中包含的正整数 **互不相同** ，另给你一个整数 `target`。
>
> 请判断是否可以将 `nums` 分成两个 **非空**、**互不相交** 的 **子集** ，并且每个元素必须  **恰好** 属于 **一个** 子集，使得这两个子集中元素的乘积都等于 `target`。
>
> 如果存在这样的划分，返回 `true`；否则，返回 `false`。
>
> **子集** 是数组中元素的一个选择集合。

---

周赛题，可以用 dfs 搜索，也可以用二进制枚举，不过都差不多

```go
func checkEqualPartitions(nums []int, target int64) bool {
    all := 1
    for _, x := range nums {
        all *= x
    }
    if int(target * target) != all {
        return false
    }
    n := len(nums)
    var dfs func(cur, st int) bool
    dfs = func(cur, st int) bool {
        if cur == int(target) {
            return true
        }
        if st == n {
            return false
        }
        for i := st; i < n; i ++ {
            cur *= nums[i]
            if dfs(cur, i + 1) {
                return true
            }
            cur /= nums[i]
        }
        return false
    }
    return dfs(1, 0)
}

```

