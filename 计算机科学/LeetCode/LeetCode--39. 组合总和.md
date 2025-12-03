[39. 组合总和](https://leetcode.cn/problems/combination-sum/)

> 给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。
>
> `candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 
>
> 对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

---

dfs直接开搜，感觉像力扣这种用全局变量也不方便还是用匿名函数方便点，byd。

```go
func combinationSum(candidates []int, target int) [][]int {
    ans := [][]int{}
    dfs(candidates, target, 0, []int{}, &ans, 0)
    return ans
}

func dfs(candidates []int, target int, sum int, curs []int, ans *[][]int, base int) {
    if sum > target {
        return
    }
    if sum == target {
        *ans = append(*ans, append([]int{}, curs...))
        return
    }
    for i := base; i < len(candidates); i ++ {
        curs = append(curs, candidates[i])
        dfs(candidates, target, sum + candidates[i], curs, ans, i)
        curs = curs[:len(curs) - 1]
    }
}
```

