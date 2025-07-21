[503. 下一个更大元素 II](https://leetcode.cn/problems/next-greater-element-ii/)

> 给定一个循环数组 `nums` （ `nums[nums.length - 1]` 的下一个元素是 `nums[0]` ），返回 *`nums` 中每个元素的 **下一个更大元素*** 。
>
> 数字 `x` 的 **下一个更大的元素** 是按数组遍历顺序，这个数字之后的第一个比它更大的数，这意味着你应该循环地搜索它的下一个更大的数。如果不存在，则输出 `-1` 。

---

依旧是栈，从后往前遍历两次就行了

```go
func nextGreaterElements(nums []int) []int {
    n := len(nums)
    ans := make([]int, n)
    for i := range ans {
        ans[i] = -1
    }
    stk := make([]int, 0)
    for i := 2 * n - 1; i >= 0; i -- {
        x := nums[i % n]
        for len(stk) > 0 && x >= stk[len(stk) - 1] {
            stk = stk[:len(stk) - 1]
        }
        if i < n && len(stk) > 0 {
            ans[i] = stk[len(stk) - 1]
        }
        stk = append(stk, x)
    }
    return ans
}
```

