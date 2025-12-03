[2799. 统计完全子数组的数目](https://leetcode.cn/problems/count-complete-subarrays-in-an-array/)

> 给你一个由 **正** 整数组成的数组 `nums` 。
>
> 如果数组中的某个子数组满足下述条件，则称之为 **完全子数组** ：
>
> - 子数组中 **不同** 元素的数目等于整个数组不同元素的数目。
>
> 返回数组中 **完全子数组** 的数目。
>
> **子数组** 是数组中的一个连续非空序列。

---

这几道题都是滑动窗口 + 向外扩展的形式，哈希表，统计不同的字符数量，然后作为 target 值，依旧滑窗。

```go
func countCompleteSubarrays(nums []int) int {
    mp := make(map[int]int)
    target := 0
    for _, x := range nums {
        if mp[x] == 0 {
            target ++
        }
        mp[x] ++
    }
    pro := make(map[int]int)
    cnt := 0
    l := 0
    ans := 0
    for r, x := range nums {
        if pro[x] == 0 {
            cnt ++
        }
        pro[x] ++
        for cnt == target {
            pro[nums[l]] --
            if pro[nums[l]] == 0 {
                cnt --
            }
            l ++
            ans += len(nums) - r
        }
    }
    return ans
}
```



