[46. 全排列](https://leetcode.cn/problems/permutations/)

>给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

----

## dfs

用一个st的布尔数组维护遍历的状态，与此同时，虽然切片是值传递，但是底层依旧共享一个数组，所以依旧需要进行深拷贝，再将拷贝的切片存入ans中

```go
func permute(nums []int) [][]int {
	n := len(nums)
	var ans [][]int
	var cur []int
	st := make([]bool, n)
	dfs(nums, n, &ans, 0, cur, st)
	return ans
}

func dfs(nums []int, n int, ans *[][]int, depth int, cur []int, st []bool) {
	if depth == n {
        tmp := make([]int, len(cur))
        copy(tmp, cur)
		*ans = append(*ans, tmp)
		return
	}
	for i := 0; i < n; i++ {
		if !st[i] {
			cur = append(cur, nums[i])
			st[i] = true
			dfs(nums, n, ans, depth+1, cur, st)
			st[i] = false
			cur = cur[:len(cur)-1]
		}
	}
}
```

