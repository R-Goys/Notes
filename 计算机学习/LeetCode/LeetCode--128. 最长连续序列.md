[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

> 给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
>
> 请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

----

原本还以为是DP，想了一会感觉是哈希表，最后还真的是，姑且也算自己想出来了。

```go
func longestConsecutive(nums []int) int {
    mp := make(map[int]bool, 1)
    ans := 0
    for i := 0; i < len(nums); i ++ {
        mp[nums[i]] = true
    }
    for num := range mp {
        if !mp[num - 1]{
            cnt := 1
            cur := num
            for mp[cur + 1] {
                cur ++
                cnt ++
            }
            ans = max(cnt, ans)
        }
    }
    return ans
}
```

