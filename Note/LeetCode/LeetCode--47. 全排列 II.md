[47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

> 给定一个可包含重复数字的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。

---

相比于全排列I，这道题显然多了一个包含重复数字的选项，这就导致去重成为了本题最大的难题，本来一开始想和之前做的那道题一样排序然后键值对统计，然后发现貌似不太行，于是还是去看了题解...

在题解中，大部分还是和全排列没啥区别，但是在判断是否跳过当前路径的的if语句多了一个条件判断`(i > 0 && nums[i] == nums[i - 1] && !st[i - 1])`这样如果是当前数字与前一个数字相同，那么只有在前面的数字被选择之后，才可以选择当前数字，以此来达到去重的目的。

```go
func permuteUnique(nums []int) [][]int {
    sort.Ints(nums)
    var ans [][]int
    var cur []int
    st := make([]bool, len(nums))
    dfs(nums, &ans, 0, cur, st)
    return ans
}

func dfs(nums []int, ans *[][]int, depth int, cur []int, st []bool) {
    if depth == len(nums) {
        tmp := make([]int, len(cur))
        copy(tmp, cur)
        *ans = append(*ans, tmp)
        return
    }
    for i := 0; i < len(nums); i ++ {
        if st[i] || (i > 0 && nums[i] == nums[i - 1] && !st[i - 1]) {
            continue
        }
        cur = append(cur, nums[i])
        st[i] = true
        dfs(nums, ans, depth + 1, cur, st)
        st[i] = false
        cur = cur[:len(cur) - 1]
    }
}
```

