[2537. 统计好子数组的数目](https://leetcode.cn/problems/count-the-number-of-good-subarrays/)

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你返回 `nums` 中 **好** 子数组的数目。
>
> 一个子数组 `arr` 如果有 **至少** `k` 对下标 `(i, j)` 满足 `i < j` 且 `arr[i] == arr[j]` ，那么称它是一个 **好** 子数组。
>
> **子数组** 是原数组中一段连续 **非空** 的元素序列。

---

这里我们通过哈希表来存储当前滑窗中每个数字出现的数量，而对应的（i，j）对数则通过 cnt 变量来存储，那么问题来了，如何去计算这个 cnt ？其实很简单，每次遍历到一个数字的时候，我们就可以通过加上这个数字对应的在哈希表中出现的次数来得到数对对应的数量。

然后滑窗

```go
func countGood(nums []int, k int) int64 {
    mp := make(map[int]int)
    l := 0
    ans := 0
    cnt := 0
    for r, x := range nums {
        cnt += mp[x]
        mp[x] ++
        for cnt >= k {
            ans += len(nums) - r
            mp[nums[l]] --
            cnt -= mp[nums[l]]
            l ++
        }
    }
    return int64(ans)
}
```



