[442. 数组中重复的数据](https://leetcode.cn/problems/find-all-duplicates-in-an-array/)

> 给你一个长度为 `n` 的整数数组 `nums` ，其中 `nums` 的所有整数都在范围 `[1, n]` 内，且每个整数出现 **最多****两次** 。请你找出所有出现 **两次** 的整数，并以数组形式返回。
>
> 你必须设计并实现一个时间复杂度为 `O(n)` 且仅使用常量额外空间（不包括存储输出所需的空间）的算法解决此问题。

---

这个b算法真的是人能想出来的，真天才

因为每个数字都在1-n之间，所以每个数都应该会存在正确的位置，当我们将每个数字放到正确的位置，其中如果有重复的数字，一定会不在这个正确的位置，此时加入我们的ans即可。

第二个for循环终止条件，其实就是在数字被移动到了正确的位置，或者说，数字重复了，这里实际上每次并不是将在当前位置拿到正确的数字，而是将正确的数字推送到正确的位置，如果相同，或者当前位置也变成正确的位置了，就没有必要继续了，前进到下一个位置。

```go
func findDuplicates(nums []int) []int {
    ans := []int{}
    for i := range nums {
        for nums[i] != nums[nums[i] - 1] {
            nums[i], nums[nums[i] - 1] = nums[nums[i] - 1], nums[i]
        }
    }
    for k, v := range nums {
        if v - 1 != k {
            ans = append(ans, v)
        }
    }
    return ans
}
```

方法二：

不得不说，这些思路是真的牛逼，为x位置的数字打上负数标记，这样，第二次遍历到相同的x，就能够直接加入ans了

```go
func findDuplicates(nums []int) []int {
    ans := []int{}
    for _, x := range nums {
        if x < 0 {
            x = -x
        }
        if nums[x - 1] > 0 {
            nums[x - 1] = - nums[x - 1]
        } else {
            ans = append(ans, x)
        }
    }
    return ans
}
```

