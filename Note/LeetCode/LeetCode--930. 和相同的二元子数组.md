[930. 和相同的二元子数组](https://leetcode.cn/problems/binary-subarrays-with-sum/)

> 给你一个二元数组 `nums` ，和一个整数 `goal` ，请你统计并返回有多少个和为 `goal` 的 **非空** 子数组。
>
> **子数组** 是数组的一段连续部分。

---

前缀和

```go
func numSubarraysWithSum(nums []int, goal int) int {
    mp := make(map[int]int)
    mp[0] = 1
    pre := 0
    ans := 0
    for _, x := range nums {
        pre += x
        ans += mp[pre - goal]
        mp[pre] ++
    }
    return ans
}
```

