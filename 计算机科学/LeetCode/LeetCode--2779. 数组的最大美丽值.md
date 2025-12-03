[2779. 数组的最大美丽值](https://leetcode.cn/problems/maximum-beauty-of-an-array-after-applying-operation/)

> 给你一个下标从 **0** 开始的整数数组 `nums` 和一个 **非负** 整数 `k` 。
>
> 在一步操作中，你可以执行下述指令：
>
> - 在范围 `[0, nums.length - 1]` 中选择一个 **此前没有选过** 的下标 `i` 。
> - 将 `nums[i]` 替换为范围 `[nums[i] - k, nums[i] + k]` 内的任一整数。
>
> 数组的 **美丽值** 定义为数组中由相等元素组成的最长子序列的长度。
>
> 对数组 `nums` 执行上述操作任意次后，返回数组可能取得的 **最大** 美丽值。
>
> **注意：**你 **只** 能对每个下标执行 **一次** 此操作。
>
> 数组的 **子序列** 定义是：经由原数组删除一些元素（也可能不删除）得到的一个新数组，且在此过程中剩余元素的顺序不发生改变。

---

主打一个暴力简单，排序 + 滑窗

```go
func maximumBeauty(nums []int, k int) int {
    sort.Ints(nums)
    l := 0
    ans := 1
    for r, x := range nums {
        for nums[l] < x - 2 * k {
            l ++
        }
        ans = max(r - l + 1, ans)
    }
    return ans
}
```

更低的时间复杂度，差分

```go
func maximumBeauty(nums []int, k int) int {
    m := 0
    for _, x := range nums {
        m = max(m, x)
    }
    diff := make([]int, m + 2)
    for _, x := range nums {
        diff[max(x - k, 0)] ++
        diff[min(x + k + 1, m + 1)] --
    }
    res, count := 0, 0
    for _, x := range diff {
        count += x
        res = max(res, count)
    }
    return res
}
```

