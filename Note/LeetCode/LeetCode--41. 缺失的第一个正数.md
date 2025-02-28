[41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

> 给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。
>
> 请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

---

第一次遍历，将所有<=0的数字设为n+1，以便不会影响之后的标记，

第二次遍历，拿到数组中每个元素的绝对值，由于我没找的是没有出现的最小正整数，所以找到的数字一定不会>=n，所以我们可以利用这种性质，将数组中每个元素的值作为**下标**，将**存在的下标**对应的数组元素设置为负数，

由于我们已经知道，数组中已经不存在负数，也就是说，如果这个下标的值为负数，那么这个下标**i + 1**一定存在于原数组中，相反，如果为正数，那么就是不存在于原数组中的，随后进行最后一层遍历，就可以拿到我们的答案了，如果遍历完了，也没有正数的话，说明n + 1就是我们最后的答案。

```go
func firstMissingPositive(nums []int) int {
    n := len(nums)
    for i := 0; i < n; i ++ {
        if nums[i] <= 0 {
            nums[i] = n + 1
        }
    }
    for i := 0; i < n; i ++ {
        num := abs(nums[i])
        if num <= n {
            nums[num - 1] = -abs(nums[num - 1])
        }
    }
    for i := 0; i < n; i ++ {
        if nums[i] > 0 {
            return i + 1
        }
    }
    return n + 1
}

func abs(a int) int {
    if a > 0 {
        return a
    }
    return -a
}
```

