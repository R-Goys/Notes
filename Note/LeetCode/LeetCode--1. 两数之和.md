[1. 两数之和](https://leetcode.cn/problems/two-sum/)

> 给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。
>
> 你可以按任意顺序返回答案。

----

## 哈希表

```go
func twoSum(nums []int, target int) []int {
    mp := make(map[int]int, 1)
    for i := 0; i < len(nums); i++ {
        j, ok := mp[target - nums[i]]
        if ok {
            return []int{i, j}
        }
        mp[nums[i]] = i
    }
    return nil
}
```

