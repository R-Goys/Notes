[78. 子集](https://leetcode.cn/problems/subsets/)

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

----

直接暴搜dfs，有两种选择，一种是选择当前数字，一种是不选择当前数字，注意搜索之后需要回溯

```go
func subsets(nums []int) [][]int {
    var ans [][]int
    var tmp []int
    dfs(nums, &ans, 0, tmp)
    return ans
}

func dfs(nums []int, ans *[][]int, begin int, tmp []int) {
    if begin == len(nums) {
        *ans = append(*ans, append([]int{},tmp...))
        return
    }
    tmp = append(tmp, nums[begin])
    dfs(nums, ans, begin + 1, tmp)
    tmp = tmp[:len(tmp) - 1]
    dfs(nums, ans, begin + 1, tmp)
}
```

